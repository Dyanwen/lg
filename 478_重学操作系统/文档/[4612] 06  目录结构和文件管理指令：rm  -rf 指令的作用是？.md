<p data-nodeid="386694" class="">通过模块一的学习，你应该掌握了计算机组成原理的重点知识，到了模块二，我们开始学习 Linux 指令，它是操作系统的前端，学好这部分内容一方面可以帮助你应对工作场景，另一方面可以让你在学习操作系统底层知识前，对 Linux 有一个大概的了解。</p>
<p data-nodeid="386695"><strong data-nodeid="386894">接下来，我们依然通过一道常见的高频面试题，引出今天的主要内容。面试题如下：请你说说</strong><code data-backticks="1" data-nodeid="386889">rm / -rf</code><strong data-nodeid="386895">的作用</strong>？</p>
<p data-nodeid="386696">相信 90% 的同学是知道这个指令的。这里先预警一下，你千万不要轻易在服务器上尝试。要想知道这条指令是做什么的，能够帮助我们解决哪些问题，那就请你认真学习今天的内容。在本课时的最后我会公布这道题目的分析过程和答案。</p>
<h3 data-nodeid="386697">什么是 Shell</h3>
<p data-nodeid="386698">在我们学习 Linux 指令之前，先来说一下什么是 Shell？Shell 把我们输入的指令，传递给操作系统去执行，所以 Shell 是一个命令行的用户界面。</p>
<p data-nodeid="386699">早期程序员没有图形界面用，就用 Shell。而且图形界面制作成本较高，不能实现所有功能，因此今天的程序员依然在用 Shell。</p>
<p data-nodeid="386700">你平时还经常会看到一个词叫作bash（Bourne Again Shell），它是用 Shell 组成的程序。这里的 Bourne 是一个人名，Steve Bourne 是 bash 的发明者。</p>
<p data-nodeid="386701">我们今天学习的所有指令，不是写死在操作系统中的，而是一个个程序。比如<code data-backticks="1" data-nodeid="386902">rm</code>指令，你可以用<code data-backticks="1" data-nodeid="386904">which</code>指令查看它所在的目录。如下图所示，你会发现<code data-backticks="1" data-nodeid="386906">rm</code>指令在<code data-backticks="1" data-nodeid="386908">/usr/bin/rm</code>目录中。</p>
<p data-nodeid="386702"><img src="https://s0.lgstatic.com/i/image/M00/56/23/Ciqc1F9rD96AC0GrAAB1NDHyN48035.png" alt="Drawing 0.png" data-nodeid="386912"></p>
<p data-nodeid="386703">如上图所示，<code data-backticks="1" data-nodeid="386914">ramroll</code>是我的英文名字，ubuntu 是我这台机器的名字。我输入了<code data-backticks="1" data-nodeid="386916">which rm</code>，然后获得了<code data-backticks="1" data-nodeid="386918">/usr/bin/rm</code>的结果，最终执行这条指令的是操作系统，连接我和操作系统的程序就是 Shell。</p>
<p data-nodeid="386704">Linux 对文件目录操作的指令就工作在 Shell 上，接下来我们讲讲文件目录操作指令。</p>
<h3 data-nodeid="386705">Linux 对文件目录的抽象</h3>
<p data-nodeid="386706">Linux 对文件进行了一个树状的抽象。<code data-backticks="1" data-nodeid="386923">/</code>代表根目录，每一节目录也用<code data-backticks="1" data-nodeid="386925">/</code>分开，所以在上图所展示的<code data-backticks="1" data-nodeid="386927">/usr/bin/rm</code>中，第一级目录是<code data-backticks="1" data-nodeid="386929">/</code>根目录，第二级目录是<code data-backticks="1" data-nodeid="386931">usr</code>目录，第三级是<code data-backticks="1" data-nodeid="386933">bin</code>目录。最后的<code data-backticks="1" data-nodeid="386935">rm</code>是一个文件。</p>
<h4 data-nodeid="386707">路径（path）</h4>
<p data-nodeid="386708">像<code data-backticks="1" data-nodeid="386939">/usr/bin/rm</code>称为可执行文件<code data-backticks="1" data-nodeid="386941">rm</code>的路径。路径就是一个文件在文件系统中的地址。如果文件系统是树形结构，那么通常一个文件只有一个地址（路径）。</p>
<p data-nodeid="386709"><strong data-nodeid="386954">目标文件的绝对路径（Absolute path），也叫作完全路径（full path），是从</strong><code data-backticks="1" data-nodeid="386946">/</code><strong data-nodeid="386955">开始，接下来每一层都是一级子目录，直到</strong>定位<strong data-nodeid="386956">到目标文件为止。</strong></p>
<p data-nodeid="386710">如上图所示的例子中，<code data-backticks="1" data-nodeid="386958">/usr/bin/rm</code>就是一个绝对路径。</p>
<h4 data-nodeid="386711">工作目录</h4>
<p data-nodeid="386712">为了方便你工作，Shell 还抽象出了工作目录。当用户打开 Shell 的时候，Shell 就会给用户安排一个工作目录。因此也就产生了相对路径。</p>
<p data-nodeid="386713">相对路径（Relative path）是以工作目录为基点的路径。比如：</p>
<ul data-nodeid="386714">
<li data-nodeid="386715">
<p data-nodeid="386716">当用户在<code data-backticks="1" data-nodeid="386964">/usr</code>目录下的时候，<code data-backticks="1" data-nodeid="386966">rm</code>文件的相对路径就是<code data-backticks="1" data-nodeid="386968">bin/rm</code>；</p>
</li>
<li data-nodeid="386717">
<p data-nodeid="386718">如果用户在<code data-backticks="1" data-nodeid="386971">/usr/bin</code>目录下的时候，<code data-backticks="1" data-nodeid="386973">rm</code>文件的路径就是<code data-backticks="1" data-nodeid="386975">./rm</code>或者<code data-backticks="1" data-nodeid="386977">rm</code>，这里用<code data-backticks="1" data-nodeid="386979">.</code>代表当前目录；</p>
</li>
<li data-nodeid="386719">
<p data-nodeid="386720">如果用户在<code data-backticks="1" data-nodeid="386982">/usr/bin/somedir</code>下，那么<code data-backticks="1" data-nodeid="386984">rm</code>的相对路径就是<code data-backticks="1" data-nodeid="386986">../rm</code>，这里用<code data-backticks="1" data-nodeid="386988">..</code>代表上一级目录。</p>
</li>
</ul>
<p data-nodeid="386721">我们使用<code data-backticks="1" data-nodeid="386991">cd</code>（change directory）指令切换工作目录，既可以用绝对路径，也可以用相对路径。 这里我要强调几个注意事项：</p>
<ul data-nodeid="386722">
<li data-nodeid="386723">
<p data-nodeid="386724">输入<code data-backticks="1" data-nodeid="386994">cd</code>，不带任何参数会切换到用户的家目录，Linux 中通常是<code data-backticks="1" data-nodeid="386996">/home/{用户名}</code>。以我自己为例，我的家目录是<code data-backticks="1" data-nodeid="386998">/home/ramroll</code>；</p>
</li>
<li data-nodeid="386725">
<p data-nodeid="386726">输入<code data-backticks="1" data-nodeid="387001">cd .</code>什么都不会发生，因为<code data-backticks="1" data-nodeid="387003">.</code>代表当前目录；</p>
</li>
<li data-nodeid="386727">
<p data-nodeid="386728">输入<code data-backticks="1" data-nodeid="387006">cd..</code>会回退一级目录，因为<code data-backticks="1" data-nodeid="387008">..</code>代表上级目录。</p>
</li>
</ul>
<p data-nodeid="386729">利用上面这 3 种能力，你就可以方便的构造相对路径了。</p>
<p data-nodeid="386730">Linux提供了一个指令<code data-backticks="1" data-nodeid="387012">pwd</code>（Print Working Directory）查看工作目录。下图是我输入<code data-backticks="1" data-nodeid="387014">pwd</code>的结果。</p>
<p data-nodeid="386731"><img src="https://s0.lgstatic.com/i/image/M00/56/23/Ciqc1F9rEAqAYNQMAACAjjKxZlw157.png" alt="Drawing 1.png" data-nodeid="387018"></p>
<p data-nodeid="386732">你可以看到我正在<code data-backticks="1" data-nodeid="387020">/home/ramroll/Documents</code>目录下工作。</p>
<h4 data-nodeid="386733">几种常见的文件类型</h4>
<p data-nodeid="386734">另一方面，Linux 下的目录也是一种文件；但是文件也不只有目录和可执行文件两种。常见的文件类型有以下 7 种:</p>
<ol data-nodeid="386735">
<li data-nodeid="386736">
<p data-nodeid="386737">普通文件（比如一个文本文件）；</p>
</li>
<li data-nodeid="386738">
<p data-nodeid="386739">目录文件（目录也是一个特殊的文件，它用来存储文件清单，比如<code data-backticks="1" data-nodeid="387026">/</code>也是一个文件）；</p>
</li>
<li data-nodeid="386740">
<p data-nodeid="386741">可执行文件（上面的<code data-backticks="1" data-nodeid="387029">rm</code>就是一个可执行文件）；</p>
</li>
<li data-nodeid="386742">
<p data-nodeid="386743">管道文件（我们会在 07 课时讨论管道文件）；</p>
</li>
<li data-nodeid="386744">
<p data-nodeid="386745">Socket 文件（我们会在模块七网络部分讨论 Socket 文件）；</p>
</li>
<li data-nodeid="386746">
<p data-nodeid="386747">软链接文件（相当于指向另一个文件所在路径的符号）；</p>
</li>
<li data-nodeid="386748">
<p data-nodeid="386749">硬链接文件（相当于指向另一个文件的指针，关于软硬链接我们将在模块六文件系统部分讨论）。</p>
</li>
</ol>
<p data-nodeid="386750">你如果使用<code data-backticks="1" data-nodeid="387036">ls -F</code>就可以看到当前目录下的文件和它的类型。比如下面这种图：</p>
<ol data-nodeid="386751">
<li data-nodeid="386752">
<p data-nodeid="386753">* 结尾的是可执行文件；</p>
</li>
<li data-nodeid="386754">
<p data-nodeid="386755">= 结尾的是 Socket 文件；</p>
</li>
<li data-nodeid="386756">
<p data-nodeid="386757">@ 结尾的是软链接；</p>
</li>
<li data-nodeid="386758">
<p data-nodeid="386759">| 结尾的管道文件；</p>
</li>
<li data-nodeid="386760">
<p data-nodeid="386761">没有符号结尾的是普通文件；</p>
</li>
<li data-nodeid="386762">
<p data-nodeid="386763">/ 结尾的是目录。</p>
</li>
</ol>
<p data-nodeid="386764"><img src="https://s0.lgstatic.com/i/image/M00/56/24/Ciqc1F9rECOAaC4iAAEqYXENnnI551.png" alt="Drawing 2.png" data-nodeid="387048"></p>
<h4 data-nodeid="386765">设备文件</h4>
<p data-nodeid="386766">Socket 是网络插座，是客户端和服务器之间同步数据的接口。其实，Linux 不只把 Socket 抽象成了文件，设备基本也都被抽象成了文件。因为设备需要不断和操作系统交换数据。而交换方式只有两种——读和写。所以设备是可以抽象成文件的，因为文件也支持这两种操作。</p>
<p data-nodeid="386767">Linux 把所有的设备都抽象成了文件，比如说打印机、USB、显卡等。这让整体的系统设计变得高度统一。</p>
<p data-nodeid="386768">至此，我们了解了 Linux 对文件目录的抽象，接下来我们看看具体的增删改查指令。</p>
<h3 data-nodeid="386769">文件的增删改查</h3>
<h4 data-nodeid="386770">增加</h4>
<p data-nodeid="441470" class="">创建一个普通文件的方法有很多，最常见的有<code data-backticks="1" data-nodeid="441472">touch</code>指令。比如下面我们创建了一个 a.txt 文件。</p>





<p data-nodeid="386772"><img src="https://s0.lgstatic.com/i/image/M00/56/2F/CgqCHl9rEC-Ae_lzAAA_P5LZwCo061.png" alt="Drawing 3.png" data-nodeid="387060"></p>
<p data-nodeid="386773"><code data-backticks="1" data-nodeid="387061">touch</code>指令本来是用来更改文件的时间戳的，但是如果文件不存在<code data-backticks="1" data-nodeid="387063">touch</code>也会帮助创建一个空文件。</p>
<p data-nodeid="386774">如果你拿到一个指令不知道该怎么用，比如<code data-backticks="1" data-nodeid="387066">touch</code>，你可以用<code data-backticks="1" data-nodeid="387068">man touch</code>去获得帮助。<code data-backticks="1" data-nodeid="387070">man</code>意思是 manual，就是说明书的意思，这里指的是系统的手册。如果你不知道<code data-backticks="1" data-nodeid="387072">man</code>是什么，也可以使用<code data-backticks="1" data-nodeid="387074">man man</code>。下图是使用<code data-backticks="1" data-nodeid="387076">man man</code>的结果：</p>
<p data-nodeid="386775"><img src="https://s0.lgstatic.com/i/image/M00/56/24/Ciqc1F9rEDqAMZ0vAAXe1wrRPf0386.png" alt="Drawing 4.png" data-nodeid="387080"></p>
<p data-nodeid="386776">另外如果我们需要增加一个目录，就需要用到<code data-backticks="1" data-nodeid="387082">mkdir</code>指令（ make directory），比如我们创建一个<code data-backticks="1" data-nodeid="387084">hello</code>目录，如下图所示：</p>
<p data-nodeid="386777"><img src="https://s0.lgstatic.com/i/image/M00/56/2F/CgqCHl9rEEGAKH5HAABgveVKHzI705.png" alt="Drawing 5.png" data-nodeid="387088"></p>
<h4 data-nodeid="386778">查看</h4>
<p data-nodeid="386779">创建之后我们可以用<code data-backticks="1" data-nodeid="387091">ls</code>指令看到这个文件，<code data-backticks="1" data-nodeid="387093">ls</code>是 list 的缩写。下面是指令 'ls' 的执行结果。</p>
<p data-nodeid="386780"><img src="https://s0.lgstatic.com/i/image/M00/56/24/Ciqc1F9rEF-AVHcKAABf8ABbQ0o651.png" alt="Drawing 6.png" data-nodeid="387101"></p>
<p data-nodeid="386781">我们看到在当前的目录下有一个<code data-backticks="1" data-nodeid="387103">a.txt</code>文件，还有一个<code data-backticks="1" data-nodeid="387105">hello</code>目录。如果你知道当前的工作目录，就可以使用<code data-backticks="1" data-nodeid="387107">pwd</code>指令。</p>
<p data-nodeid="386782">如果想看到<code data-backticks="1" data-nodeid="387110">a.txt</code>更完善的信息，还可以使用<code data-backticks="1" data-nodeid="387112">ls -l</code>。<code data-backticks="1" data-nodeid="387114">-l</code>是<code data-backticks="1" data-nodeid="387116">ls</code>指令的可选参数。下图是<code data-backticks="1" data-nodeid="387118">ls -l</code>的结果，你可以看到<code data-backticks="1" data-nodeid="387120">a.txt</code>更详细的描述。</p>
<p data-nodeid="386783"><img src="https://s0.lgstatic.com/i/image/M00/56/2F/CgqCHl9rEGqAA0XWAAEv83hemN0703.png" alt="Drawing 7.png" data-nodeid="387124"></p>
<p data-nodeid="386784">如上图所示，我们看到两个<code data-backticks="1" data-nodeid="387126">ramroll</code>，它们是<code data-backticks="1" data-nodeid="387128">a.txt</code>所属的用户和所属的用户分组，刚好重名了。<code data-backticks="1" data-nodeid="387130">Sep 13</code>是日期。 中间有一个<code data-backticks="1" data-nodeid="387132">0</code>是<code data-backticks="1" data-nodeid="387134">a.txt</code>的文件大小，目前<code data-backticks="1" data-nodeid="387136">a.txt</code>中还没有写入内容，因此大小是<code data-backticks="1" data-nodeid="387138">0</code>。</p>
<p data-nodeid="386785">另外虽然<code data-backticks="1" data-nodeid="387141">hello</code>是空的目录，但是目录文件 Linux 上来就分配了<code data-backticks="1" data-nodeid="387143">4096</code>字节的空间。这是因为目录内需要保存很多文件的描述信息。</p>
<h4 data-nodeid="386786">删除</h4>
<p data-nodeid="386787">如果我们想要删除<code data-backticks="1" data-nodeid="387147">a.txt</code>可以用<code data-backticks="1" data-nodeid="387149">rm a.txt</code>；如我们要删除<code data-backticks="1" data-nodeid="387151">hello</code>目录，可以用<code data-backticks="1" data-nodeid="387153">rm hello</code>。<code data-backticks="1" data-nodeid="387155">rm</code>是 remove 的缩写。</p>
<p data-nodeid="386788"><img src="https://s0.lgstatic.com/i/image/M00/56/2F/CgqCHl9rEHSAaCuvAACmYor8yvE702.png" alt="Drawing 8.png" data-nodeid="387159"></p>
<p data-nodeid="386789">但是当我们输入<code data-backticks="1" data-nodeid="387161">rm hello</code>的时候，会提示<code data-backticks="1" data-nodeid="387163">hello</code>是一个目录，不可以删除。因此我们需要增加一个可选项，比如<code data-backticks="1" data-nodeid="387165">-r</code>即 recursive（递归）。目录是一个递归结构，所以需要用递归删除。最后，你会发现<code data-backticks="1" data-nodeid="387167">rm hello -r</code>删除了<code data-backticks="1" data-nodeid="387169">hello</code>目录。</p>
<p data-nodeid="386790">接下来我们尝试在 hello 目录下新增一个文件，比如相对路径是<code data-backticks="1" data-nodeid="387172">hello/world/os.txt</code>。需要先创建 hello/world 目录。这种情况会用到<code data-backticks="1" data-nodeid="387174">mkdir</code>的<code data-backticks="1" data-nodeid="387176">-p</code>参数，这个参数控制<code data-backticks="1" data-nodeid="387178">mkdir</code>当发现目标目录的父级目录不存在的时候会递归的创建。以下是我们的执行结果：</p>
<p data-nodeid="386791"><img src="https://s0.lgstatic.com/i/image/M00/56/24/Ciqc1F9rEJKAYE8qAAFVKf9hzs8021.png" alt="Drawing 9.png" data-nodeid="387182"></p>
<h4 data-nodeid="386792">修改</h4>
<p data-nodeid="386793">如果需要修改一个文件，可以使用<code data-backticks="1" data-nodeid="387185">nano</code>或者<code data-backticks="1" data-nodeid="387187">vi</code>编辑器。类似的工具还有很多，但是<code data-backticks="1" data-nodeid="387189">nano</code>和<code data-backticks="1" data-nodeid="387191">vi</code>一般是<code data-backticks="1" data-nodeid="387193">linux</code>自带的。</p>
<p data-nodeid="386794">这里我不展开讲解了，你可以自己去尝试。在尝试的过程中如果遇到什么问题，可以写在留言区，我会逐一为你解答。</p>
<h3 data-nodeid="386795">查阅文件内容</h3>
<p data-nodeid="386796">在了解了文件的增删改查操作后，下面我们来学习查阅文件内容。我们知道，Linux 下查阅文件内容，可以根据不同场景选择不同的指令。</p>
<p data-nodeid="386797">当文件较小时，比如一个配置文件，想要快速浏览这个文件，可以用<code data-backticks="1" data-nodeid="387199">cat</code>指令。下面 cat 指令帮助我们快速查看<code data-backticks="1" data-nodeid="387201">/etc/hosts</code>文件。<code data-backticks="1" data-nodeid="387203">cat</code>指令将文件连接到标准输出流并打印到屏幕上。</p>
<p data-nodeid="386798"><img src="https://s0.lgstatic.com/i/image/M00/56/30/CgqCHl9rEKSAetBpAAJKmXNMtek042.png" alt="Drawing 10.png" data-nodeid="387207"></p>
<p data-nodeid="399069" class="">标准输出流（Standard Output）也是一种文件，进程可以将要输出的内容写入标准输出流文件，这样就可以在屏幕中打印。</p>





<p data-nodeid="386800">如果用<code data-backticks="1" data-nodeid="387210">cat</code>查看大文件，比如一个线上的日志文件，因为动辄有几个 G，控制台打印出所有的内容就要非常久，而且刷屏显示看不到东西。</p>
<p data-nodeid="391789" class="">而且如果在线上进行查看大文件的操作，会带来不必要的麻烦：</p>




<p data-nodeid="386802">首先因为我们需要把文件拷贝到输入输出流，这需要花费很长时间，这个过程会占用机器资源；</p>
<p data-nodeid="386803">其次，本身文件会读取到内存中，这时内存被大量占用，很危险，这可能导致其他应用内存不足。因此我们需要一些不用加载整个文件，就能查看文件内容的指令。</p>
<p data-nodeid="386804"><strong data-nodeid="387218">more</strong></p>
<p data-nodeid="419518" class=""><code data-backticks="1" data-nodeid="419519">more</code>可以帮助我们读取文件，但不需要读取整个文件到内存中。本身<code data-backticks="1" data-nodeid="419521">more</code>的定位是一个阅读过滤器，比如你在<code data-backticks="1" data-nodeid="419523">more</code>里除了可以向下翻页，还可以输入一段文本进行搜索。</p>














<p data-nodeid="386806"><img src="https://s0.lgstatic.com/i/image/M00/56/30/CgqCHl9rEK6ANctWAAvN_sMIYLA038.png" alt="Drawing 11.png" data-nodeid="387227"></p>
<p data-nodeid="423911" class="">如上图所示，我在<code data-backticks="1" data-nodeid="423913">more</code>查看一个 nginx 日志后，先输入一个<code data-backticks="1" data-nodeid="423915">/</code>，然后输入<code data-backticks="1" data-nodeid="423917">192.168</code>看到的结果。<code data-backticks="1" data-nodeid="423919">more</code>帮我找到了<code data-backticks="1" data-nodeid="423921">192.168</code>所在的位置，然后又帮我定位到了这个位置。整个过程 more 指令只读取我们需要的部分到内存中。</p>



<p data-nodeid="386808"><strong data-nodeid="387242">less</strong></p>
<p data-nodeid="386809"><code data-backticks="1" data-nodeid="387243">less</code>是一个和<code data-backticks="1" data-nodeid="387245">more</code>功能差不多的工具，打开<code data-backticks="1" data-nodeid="387247">man</code>能够看到<code data-backticks="1" data-nodeid="387249">less</code>的介绍上写着自己是<code data-backticks="1" data-nodeid="387251">more</code>的反义词（opposite of more）。这样你可以看出<code data-backticks="1" data-nodeid="387253">linux</code>生态其实也是很自由的一个生态，在这里创造工具也可以按照自己的喜好写文档。<code data-backticks="1" data-nodeid="387255">less</code>支持向上翻页，这个功能<code data-backticks="1" data-nodeid="387257">more</code>是做不到的。所以现在<code data-backticks="1" data-nodeid="387259">less</code>用得更多一些。</p>
<p data-nodeid="386810"><strong data-nodeid="387264">head/tail</strong></p>
<p data-nodeid="386811"><code data-backticks="1" data-nodeid="387265">head</code>和<code data-backticks="1" data-nodeid="387267">tail</code>是一组，它们用来读取一个文件的头部 N 行或者尾部 N 行。比如一个线上的大日志文件，当线上出了 bug，服务暂停的时候，我们就可以用<code data-backticks="1" data-nodeid="387269">tail -n 1000</code>去查看最后的 1000 行日志文件，寻找导致服务异常的原因。</p>
<p data-nodeid="386812">另一个比较重要的用法是，如果你想看一个实时的<code data-backticks="1" data-nodeid="387272">nginx</code>日志，可以使用<code data-backticks="1" data-nodeid="387274">tail -f 文件名</code>，这样你会看到用户的请求不断进来。查一下<code data-backticks="1" data-nodeid="387276">man</code>，你会发现<code data-backticks="1" data-nodeid="387278">-f</code>是 follow 的意思，就是文件追加的内容会跟随输出到标准输出流。</p>
<p data-nodeid="386813"><strong data-nodeid="387283">grep</strong></p>
<p data-nodeid="386814">有时候你需要查看一个指定<code data-backticks="1" data-nodeid="387285">ip</code>的nginx日志，或者查看一段时间内的<code data-backticks="1" data-nodeid="387287">nginx</code>日志。如果不想用<code data-backticks="1" data-nodeid="387289">less</code>和<code data-backticks="1" data-nodeid="387291">more</code>进入文件中去查看，就可以用<code data-backticks="1" data-nodeid="387293">grep</code>命令。Linux 的文件命名风格都很短，所以也影响了很多人，比如之前我看到过一个大牛的程序，变量名从来不超过 5 个字母，而且都有意义。</p>
<p data-nodeid="386815">grep 这个词，我们分成三段来看，是 g|re|p。</p>
<ul data-nodeid="386816">
<li data-nodeid="386817">
<p data-nodeid="386818">g 就是 global，全局；</p>
</li>
<li data-nodeid="386819">
<p data-nodeid="386820">re 就是 regular expression，正则表达式；</p>
</li>
<li data-nodeid="386821">
<p data-nodeid="386822">p 就是 pattern，模式。</p>
</li>
</ul>
<p data-nodeid="386823">所以这个指令的作用是通过正则表达式全局搜索一个文件找到匹配的模式。我觉得这种命名真的很牛，软件命名也是一个世纪难题，grep这个名字不但发音不错，而且很有含义，又避免了名字过长，方便记忆。</p>
<p data-nodeid="386824">下面我们举两个例子看看 grep 的用法：</p>
<ul data-nodeid="386825">
<li data-nodeid="386826">
<p data-nodeid="386827">例 1：查找 ip 地址</p>
</li>
</ul>
<p data-nodeid="429757" class="">我们可以通过<code data-backticks="1" data-nodeid="429759">grep</code>命令定位某个<code data-backticks="1" data-nodeid="429761">ip</code>地址的用户都做了什么事情，如下图所示：</p>




<p data-nodeid="386829"><img src="https://s0.lgstatic.com/i/image/M00/56/30/CgqCHl9rELqAbYi4AAfJLxM4xgw204.png" alt="Drawing 12.png" data-nodeid="387313"></p>
<ul data-nodeid="386830">
<li data-nodeid="386831">
<p data-nodeid="386832">例 2：查找时间段的日志</p>
</li>
</ul>
<p data-nodeid="386833">我们可以通过 grep 命令查找某个时间段内用户都做了什么事情。如下图所示，你可以看到在某个 5 分钟内所有用户的访问情况。</p>
<p data-nodeid="386834"><img src="https://s0.lgstatic.com/i/image/M00/56/25/Ciqc1F9rEMGAQTTHAAYTLdI_HSA050.png" alt="Drawing 13.png" data-nodeid="387318"></p>
<h3 data-nodeid="386835">查找文件</h3>
<p data-nodeid="386836">用户经常还会有一种诉求，就是查找文件。</p>
<p data-nodeid="386837">之前我们使用过一个<code data-backticks="1" data-nodeid="387322">which</code>指令，这个指令可以查询一个指令文件所在的位置，比如<code data-backticks="1" data-nodeid="387324">which grep</code>会，你会看到<code data-backticks="1" data-nodeid="387326">grep</code>指令被安装的位置是<code data-backticks="1" data-nodeid="387328">/usr/bin</code>。但是我们还需要一个更加通用的指令查找文件，也就是 find 指令。</p>
<p data-nodeid="386838"><strong data-nodeid="387333">find</strong></p>
<p data-nodeid="386839">find 指令帮助我们在文件系统中查找文件。 比如我们如果想要查找所有<code data-backticks="1" data-nodeid="387335">.txt</code> 扩展名的文件，可以使用<code data-backticks="1" data-nodeid="387337">find / -iname "*.txt"</code>，<code data-backticks="1" data-nodeid="387339">-iname</code>这个参数是用来匹配查找的，i 字母代表忽略大小写，这里也可以用<code data-backticks="1" data-nodeid="387341">-name</code>替代。输入这条指令，你会看到不断查找文件，如下图所示：</p>
<p data-nodeid="386840"><img src="https://s0.lgstatic.com/i/image/M00/56/30/CgqCHl9rEM2AD9SWAAdsfnMr8fw422.png" alt="Drawing 14.png" data-nodeid="387345"></p>
<h3 data-nodeid="386841">总结</h3>
<p data-nodeid="386842">这节课我们学习了很多指令，不知道你记住了多少？最后，我们再一起复习一下。</p>
<ul data-nodeid="431217">
<li data-nodeid="431218">
<p data-nodeid="431219"><code data-backticks="1" data-nodeid="431242">pwd</code>指令查看工作目录。</p>
</li>
<li data-nodeid="431220">
<p data-nodeid="431221"><code data-backticks="1" data-nodeid="431244">cd</code>指令切换工作目录。</p>
</li>
<li data-nodeid="431222">
<p data-nodeid="431223"><code data-backticks="1" data-nodeid="431246">which</code>指令查找一个执行文件所在的路径。</p>
</li>
<li data-nodeid="431224">
<p data-nodeid="431225"><code data-backticks="1" data-nodeid="431248">ls</code>显示文件信息。</p>
</li>
<li data-nodeid="431226">
<p data-nodeid="431227" class=""><code data-backticks="1" data-nodeid="431250">rm</code>删除文件。</p>
</li>
<li data-nodeid="431228">
<p data-nodeid="431229"><code data-backticks="1" data-nodeid="431252">touch</code>修改一个文件的时间戳，如果文件不存在会触发创建文件。</p>
</li>
<li data-nodeid="431230">
<p data-nodeid="431231"><code data-backticks="1" data-nodeid="431254">vi</code>和<code data-backticks="1" data-nodeid="431256">nano</code>可以用来编辑文件。</p>
</li>
<li data-nodeid="431232">
<p data-nodeid="431233"><code data-backticks="1" data-nodeid="431258">cat</code>查看完成的文件适合小型文件。</p>
</li>
<li data-nodeid="431234">
<p data-nodeid="431235"><code data-backticks="1" data-nodeid="431260">more``less</code>查看一个文件但是只读取用户看到的内容到内存，因此消耗资源较少，适合在服务器上看日志。</p>
</li>
<li data-nodeid="431236">
<p data-nodeid="431237"><code data-backticks="1" data-nodeid="431262">head``tail</code>可以用来看文件的头和尾。</p>
</li>
<li data-nodeid="431238">
<p data-nodeid="431239"><code data-backticks="1" data-nodeid="431264">grep</code>指令搜索文件内容。</p>
</li>
<li data-nodeid="431240">
<p data-nodeid="431241"><code data-backticks="1" data-nodeid="431266">find</code>指令全局查找文件。</p>
</li>
</ul>

<p data-nodeid="434178" class="">在这里，我再强调一个指令，即<code data-backticks="1" data-nodeid="434180">man</code>指令，它是所有指令的手册，所以你一定要多多运用，熟练掌握。另外，一个指令通常有非常多的参数，但都需要用<code data-backticks="1" data-nodeid="434182">man</code>指令去仔细研究。</p>

<p data-nodeid="386869"><strong data-nodeid="387386">那么通过这节课的学习，你现在可以来回答本节关联的面试题目：</strong><code data-backticks="1" data-nodeid="387382">rm / -rf</code><strong data-nodeid="387387">的作用是？</strong></p>
<p data-nodeid="432722" class="">老规矩，请你先在脑海里先思考你的答案，并把你的思考写在留言区，然后再来看我接下来的分析。</p>

<p data-nodeid="386871"><strong data-nodeid="387392">【解析】</strong></p>
<ul data-nodeid="386872">
<li data-nodeid="386873">
<p data-nodeid="386874"><code data-backticks="1" data-nodeid="387393">/</code>是文件系统根目录；</p>
</li>
<li data-nodeid="386875">
<p data-nodeid="386876"><code data-backticks="1" data-nodeid="387395">rm</code>是删除指令；</p>
</li>
<li data-nodeid="386877">
<p data-nodeid="386878"><code data-backticks="1" data-nodeid="387397">-r</code>是 recursive（递归）；</p>
</li>
<li data-nodeid="386879">
<p data-nodeid="386880"><code data-backticks="1" data-nodeid="387399">-f</code>是 force（强制），遇到只读文件也不提示，直接删除。</p>
</li>
</ul>
<p data-nodeid="386881">所以<code data-backticks="1" data-nodeid="387402">rm -rf /</code>就是删除整个文件系统上的所有文件，而且不用给用户提示。</p>
<h3 data-nodeid="386882">课后习题</h3>
<p data-nodeid="386883"><strong data-nodeid="387417">最后再给你出一道查资料的面试题，搜索文件系统中所有以包含</strong><code data-backticks="1" data-nodeid="387408">std</code><strong data-nodeid="387418">字符串且以</strong><code data-backticks="1" data-nodeid="387412">.h</code><strong data-nodeid="387419">扩展名结尾的文件</strong>。</p>
<p data-nodeid="386884" class="">你可以把你的答案、思路或者课后总结写在留言区，这样可以帮助你产生更多的思考，这也是构建知识体系的一部分。经过长期的积累，相信你会得到意想不到的收获。如果你觉得今天的内容对你有所启发，欢迎分享给身边的朋友。期待看到你的思考！</p>

---

### 精选评论

##### 林䭽：
> 大家可以的。

##### *方：
> find / -iname “*std*.h”

##### *达：
> 原来grep是global regular pattern的组合，学到了

##### **伟：
> 当初在学校学的时候没认真学，现在出来工作了才知道有多当时学的有重要。

##### *邦：
> rm -rf *

##### 曙：
> sudo find / -type f -name "*std*.h"

##### **3654：
> find / -name '*.h'|grep -rl 'std'

##### **用户7573：
> rm -rf /* 表示清空服务器的缓存垃圾，当服务器使用长时间没有关机时，记得定期执行一下这个命令，清空一下缓存呀，清空完垃圾后，你会发现服务器就和起飞一样，蹭蹭蹭蹭的快。😉

##### **俊：
> find / -name "*std*.h"

##### **民：
> sudo find / -name "*.sh" -name "*start*"

##### **峰：
> 请问标准输出流文件是哪个文件，一个进程是如何将数据输出至屏幕的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在模块六：文件系统部分，会详细讨论这个内容。同学如果着急了解，可以在网上查查资料

##### *凡：
> 很棒，要多看几遍

##### **达：
> 课后习题：find / -iname "*std*.py"

##### *洋：
> find -name "*.h"|xargs grep "std" -l

##### **煌：
> find / -iname "*std*.h"

##### **宇：
> find / -name "*std*.h"

##### **朝：
> find / -type f -name "*std*.h"

##### hamburgduo：
> 写的真好！

##### **斌：
> find / -name “*.h” -a -name “*std*”😀

##### *潮：
> find -name "*std*.h"

##### *兴：
> find / -iname "*std*.h"

##### **龙：
> find / -iname "*std*.h"

##### **1873：
> Awk sed Greg这三个命令在脚本中经常看到

##### *哲：
> sudo find / -name "*std*.h"

##### **鹏：
> find / -type f -name "*.h" | xargs grep -in "std"

