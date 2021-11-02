<p data-nodeid="2209" class="">在面试中，我们经常会遇到面试官询问 Linux 指令，06 课时中讲到的<code data-backticks="1" data-nodeid="2358">rm -rf /</code>属于比较简单的题目，相当于小学难度。<strong data-nodeid="2368">这节课给你带来一道初中难度的题目：</strong><code data-backticks="1" data-nodeid="2363">xargs</code><strong data-nodeid="2369">指令的作用是什么</strong>？</p>
<p data-nodeid="2210">通常这个指令是和管道一起使用，因此就引出了这节课的主题：管道。为了理解管道，和学习管道相关的内容，还有一些概念需要你理解，比如：进程、标准流和重定向。好的，接下来请和我一起，把这块知识一网打尽！</p>
<h3 data-nodeid="2211">进程</h3>
<p data-nodeid="2212">为了弄清楚这节课程的内容，也就是管道，我们先来讨论一下进程。</p>
<p data-nodeid="2213">我们知道，应用的可执行文件是放在文件系统里，把可执行文件启动，就会在操作系统里（具体来说是内存中）形成一个应用的副本，这个副本就是进程。</p>
<p data-nodeid="2214"><em data-nodeid="2378"><strong data-nodeid="2377">插一个小知识，以后你再遇到面试题：什么是进程？</strong></em></p>
<p data-nodeid="2215"><i><strong data-nodeid="2387">可以回答：进程是应用的执行副本；而不要回答进程是操作系统分配资源的最小单位。前者是定义，后者是作用</strong></i>*。*</p>
<p data-nodeid="2216"><strong data-nodeid="2391">ps</strong></p>
<p data-nodeid="2217">如果你要看当前的进程，可以用<code data-backticks="1" data-nodeid="2393">ps</code>指令。p 代表 processes，也就是进程；s 代表 snapshot，也就是快照。所谓快照，就是像拍照一样。</p>
<p data-nodeid="2218"><img src="https://s0.lgstatic.com/i/image/M00/58/00/CgqCHl9twJSAZMbHAADiXX3JRVw649.png" alt="Drawing 0.png" data-nodeid="2397"></p>
<p data-nodeid="2219">如上图所示，我启动了两个进程，<code data-backticks="1" data-nodeid="2399">ps</code>和<code data-backticks="1" data-nodeid="2401">bash</code>。ps 就是我刚刚启动的，被<code data-backticks="1" data-nodeid="2403">ps</code>自己捕捉到了；<code data-backticks="1" data-nodeid="2405">bash</code>是因为我开了这个控制台，执行的<code data-backticks="1" data-nodeid="2407">shell</code>是<code data-backticks="1" data-nodeid="2409">bash</code>。</p>
<p data-nodeid="2220">当然操作系统也不可能只有这么几个进程，这是因为不带任何参数的<code data-backticks="1" data-nodeid="2412">ps</code>指令显示的是同一个电传打字机（TTY上）的进程。TTY 这个概念是一个历史的概念，过去用来传递信息，现在已经被传真、邮件、微信等取代。</p>
<p data-nodeid="2221">操作系统上的 TTY 是一个输入输出终端的概念，比如用户打开 bash，操作系统就为用户分配了一个输入输出终端。没有加任何参数的<code data-backticks="1" data-nodeid="2415">ps</code>只显示在同一个 TTY 的进程。</p>
<p data-nodeid="2222">如果想看到所有的进程，可以用<code data-backticks="1" data-nodeid="2418">ps -e</code>，<code data-backticks="1" data-nodeid="2420">-e</code>没有特殊含义，只是为了和<code data-backticks="1" data-nodeid="2422">-A</code>区分开。我们通常不直接用<code data-backticks="1" data-nodeid="2424">ps -e</code>而是用<code data-backticks="1" data-nodeid="2426">ps -ef</code>，这是因为<code data-backticks="1" data-nodeid="2428">-f</code>可以带上更多的描述字段，如下图所示：</p>
<p data-nodeid="2223"><img src="https://s0.lgstatic.com/i/image/M00/57/F5/Ciqc1F9twKuAcx9KAAMttqMWk0U603.png" alt="Drawing 1.png" data-nodeid="2432"></p>
<ul data-nodeid="2224">
<li data-nodeid="2225">
<p data-nodeid="2226">UID 指进程的所有者；</p>
</li>
<li data-nodeid="2227">
<p data-nodeid="2228">PID 是进程的唯一标识；</p>
</li>
<li data-nodeid="2229">
<p data-nodeid="2230">PPID 是进程的父进程 ID；</p>
</li>
<li data-nodeid="2231">
<p data-nodeid="2232">C 是 CPU 的利用率（就是 CPU 占用）；</p>
</li>
<li data-nodeid="2233">
<p data-nodeid="2234">STIME 是开始时间；</p>
</li>
<li data-nodeid="2235">
<p data-nodeid="2236">TTY 是进程所在的 TTY，如果没有 TTY 就是 ？号；</p>
</li>
<li data-nodeid="2237">
<p data-nodeid="2238">TIME；</p>
</li>
<li data-nodeid="2239">
<p data-nodeid="2240">CMD 是进程启动时的命令，如果不是一个 Shell 命令，而是用方括号括起来，那就是系统进程或者内核过程。</p>
</li>
</ul>
<p data-nodeid="2241">另外一个用得比较多的是<code data-backticks="1" data-nodeid="2442">ps aux</code>，它和<code data-backticks="1" data-nodeid="2444">ps -ef</code>能力差不多，但是是 BSD 风格的。就是加州伯克利分校研发的 Unix 分支版本的衍生风格，这种风格其实不太好描述，我截了一张图，你可以体会一下：</p>
<p data-nodeid="2242"><img src="https://s0.lgstatic.com/i/image/M00/58/00/CgqCHl9twMGAAl8XAAOd-4G_G6U649.png" alt="Drawing 2.png" data-nodeid="2448"></p>
<p data-nodeid="2243">在 BSD 风格中有些字段的叫法和含义变了，如果你感兴趣，可以作为课后延伸学习的内容。</p>
<h4 data-nodeid="2244">top</h4>
<p data-nodeid="2245">另外还有一个和<code data-backticks="1" data-nodeid="2452">ps</code>能力差不多，但是显示的不是快照而是实时更新数据的<code data-backticks="1" data-nodeid="2454">top</code>指令。因为自带的<code data-backticks="1" data-nodeid="2456">top</code>显示的内容有点少， 所以我喜欢用一个叫作<code data-backticks="1" data-nodeid="2458">htop</code>的指令，具体的安装全方法我会在 10 | 软件的安装： 编译安装和包管理器安装有什么优势和劣势？中给你介绍。本课时，我们先看一下使用效果，如下图所示：</p>
<p data-nodeid="2246"><img src="https://s0.lgstatic.com/i/image/M00/57/F5/Ciqc1F9twNKAbWUxAAjBKXaXn90775.png" alt="Drawing 3.png" data-nodeid="2464"></p>
<p data-nodeid="2247">以上，我们一起把进程学了一个皮毛，更多关于进程的内容我们会在模块四：进程和线程中讨论。</p>
<h3 data-nodeid="2248">管道（Pipeline）</h3>
<p data-nodeid="2249">现在你已经掌握了一点点进程的基础，下面我们来学习管道，管道（Pipeline）的作用是在命令和命令之间，传递数据。比如说一个命令的结果，就可以作为另一个命令的输入。我们了解了进程，所以这里说的命令就是进程。更准确地说，管道在进程间传递数据。</p>
<h4 data-nodeid="2250">输入输出流</h4>
<p data-nodeid="2251">每个进程拥有自己的标准输入流、标准输出流、标准错误流。</p>
<p data-nodeid="2252">这几个标准流说起来很复杂，但其实都是文件。</p>
<ul data-nodeid="2253">
<li data-nodeid="2254">
<p data-nodeid="2255">标准输入流（用 0 表示）可以作为进程执行的上下文（进程执行可以从输入流中获取数据）。</p>
</li>
<li data-nodeid="2256">
<p data-nodeid="2257">标准输出流（用 1 表示）中写入的结果会被打印到屏幕上。</p>
</li>
<li data-nodeid="2258">
<p data-nodeid="2259">如果进程在执行过程中发生异常，那么异常信息会被记录到标准错误流（用 2 表示）中。</p>
</li>
</ul>
<p data-nodeid="2260"><strong data-nodeid="2477">重定向</strong></p>
<p data-nodeid="2261">我们执行一个指令，比如<code data-backticks="1" data-nodeid="2479">ls -l</code>，结果会写入标准输出流，进而被打印。这时可以用重定向符将结果重定向到一个文件，比如说<code data-backticks="1" data-nodeid="2481">ls -l &gt; out</code>，这样<code data-backticks="1" data-nodeid="2483">out</code>文件就会有<code data-backticks="1" data-nodeid="2485">ls -l</code>的结果；而屏幕上也不会再打印<code data-backticks="1" data-nodeid="2487">ls -l</code>的结果。</p>
<p data-nodeid="2262"><img src="https://s0.lgstatic.com/i/image/M00/57/F5/Ciqc1F9twOiAWhAGAAU25o5Gb_s323.png" alt="Drawing 4.png" data-nodeid="2491"></p>
<p data-nodeid="2263">具体来说<code data-backticks="1" data-nodeid="2493">&gt;</code>符号叫作覆盖重定向；<code data-backticks="1" data-nodeid="2495">&gt;&gt;</code>叫作追加重定向。<code data-backticks="1" data-nodeid="2497">&gt;</code>每次都会把目标文件覆盖，<code data-backticks="1" data-nodeid="2499">&gt;&gt;</code>会在目标文件中追加。比如你每次启动一个程序日志都写入<code data-backticks="1" data-nodeid="2501">/var/log/somelogfile</code>中，可以这样操作，如下所示：</p>
<pre class="lang-java" data-nodeid="2264"><code data-language="java">start.sh &gt;&gt; /<span class="hljs-keyword">var</span>/log/somelogfile
</code></pre>
<p data-nodeid="2265">经过这样的操作后，每次执行程序日志就不会被覆盖了。</p>
<p data-nodeid="2266">另外还有一种情况，比如我们输入:</p>
<pre class="lang-java" data-nodeid="2267"><code data-language="java">ls1 &gt; out
</code></pre>
<p data-nodeid="2268">结果并不会存入<code data-backticks="1" data-nodeid="2506">out</code>文件，因为<code data-backticks="1" data-nodeid="2508">ls1</code>指令是不存在的。结果会输出到标准错误流中，仍然在屏幕上。这里我们可以把标准错误流也重定向到标准输出流，然后再重定向到文件。</p>
<pre class="lang-java" data-nodeid="2269"><code data-language="java">ls1 &amp;&gt; out
</code></pre>
<p data-nodeid="2270">这个写法等价于：</p>
<pre class="lang-java" data-nodeid="2271"><code data-language="java">ls1 &gt; out <span class="hljs-number">2</span>&gt;&amp;<span class="hljs-number">1</span>
</code></pre>
<p data-nodeid="2272"><img src="https://s0.lgstatic.com/i/image/M00/58/00/CgqCHl9twP2AefIFAAL1fMsTbHk961.png" alt="Drawing 5.png" data-nodeid="2513"></p>
<p data-nodeid="2273">相当于把<code data-backticks="1" data-nodeid="2515">ls1</code>的标准输出流重定向到<code data-backticks="1" data-nodeid="2517">out</code>，因为<code data-backticks="1" data-nodeid="2519">ls1 &gt; out</code>出错了，所以标准错误流被定向到了标准输出流。<code data-backticks="1" data-nodeid="2521">&amp;</code>代表一种引用关系，具体代表的是<code data-backticks="1" data-nodeid="2523">ls1 &gt;out</code>的标准输出流。</p>
<h4 data-nodeid="2274">管道的作用和分类</h4>
<p data-nodeid="2275">有了进程和重定向的知识，接下来我们梳理下管道的作用。管道（Pipeline）将一个进程的输出流定向到另一个进程的输入流，就像水管一样，作用就是把这两个文件接起来。如果一个进程输出了一个字符 X，那么另一个进程就会获得 X 这个输入。</p>
<p data-nodeid="2276"><strong data-nodeid="2531">管道和重定向很像，但是管道是一个连接一个进行计算，重定向是将一个文件的内容定向到另一个文件，这二者经常会结合使用</strong>。</p>
<p data-nodeid="2277">Linux 中的管道也是文件，有两种类型的管道：</p>
<ol data-nodeid="2278">
<li data-nodeid="2279">
<p data-nodeid="2280">匿名管道（Unnamed Pipeline），这种管道也在文件系统中，但是它只是一个存储节点，不属于任何一个目录。说白了，就是没有路径。</p>
</li>
<li data-nodeid="2281">
<p data-nodeid="2282">命名管道（Named Pipeline），这种管道就是一个文件，有自己的路径。</p>
</li>
</ol>
<h4 data-nodeid="2283">FIFO</h4>
<p data-nodeid="2284">管道具有 FIFO（First In First Out），FIFO 和排队场景一样，先排到的先获得。所以先流入管道文件的数据，也会先流出去传递给管道下游的进程。</p>
<h3 data-nodeid="2285">使用场景分析</h3>
<p data-nodeid="2286">接下来我们以多个场景举例帮助你深入学习管道。</p>
<h4 data-nodeid="2287">排序</h4>
<p data-nodeid="2288">比如我们用<code data-backticks="1" data-nodeid="2541">ls</code>，希望按照文件名排序倒序，可以使用匿名管道，将<code data-backticks="1" data-nodeid="2543">ls</code>的结果传递给<code data-backticks="1" data-nodeid="2545">sort</code>指令去排序。你看，这样<code data-backticks="1" data-nodeid="2547">ls</code>的开发者就不用关心排序问题了。</p>
<p data-nodeid="2289"><img src="https://s0.lgstatic.com/i/image/M00/57/F5/Ciqc1F9twQmAUpYzAADI43WGK9A660.png" alt="Drawing 6.png" data-nodeid="2551"></p>
<h4 data-nodeid="2290">去重</h4>
<p data-nodeid="2291">另一个比较常见的场景是去重，比如有一个字典文件，里面都是词语。如下所示：</p>
<pre class="lang-java" data-nodeid="2292"><code data-language="java">Apple
Banana
Apple
Banana
……
</code></pre>
<p data-nodeid="2293">如果我们想要去重可以使用<code data-backticks="1" data-nodeid="2555">uniq</code>指令，<code data-backticks="1" data-nodeid="2557">uniq</code>指令能够找到文件中相邻的重复行，然后去重。但是我们上面的文件重复行是交替的，所以不可以直接用<code data-backticks="1" data-nodeid="2559">uniq</code>，因此可以先<code data-backticks="1" data-nodeid="2561">sort</code>这个文件，然后利用管道将<code data-backticks="1" data-nodeid="2563">sort</code>的结果重定向到<code data-backticks="1" data-nodeid="2565">uniq</code>指令。指令如下：</p>
<p data-nodeid="2294"><img src="https://s0.lgstatic.com/i/image/M00/58/00/CgqCHl9twRGAXmhPAACPjv2JnVo451.png" alt="Drawing 7.png" data-nodeid="2569"></p>
<h4 data-nodeid="2295">筛选</h4>
<p data-nodeid="2296">有时候我们想根据正则模式筛选对应的内容。比如说我们想找到项目文件下所有文件名中含有<code data-backticks="1" data-nodeid="2572">Spring</code>的文件。就可以利用<code data-backticks="1" data-nodeid="2574">grep</code>指令，操作如下：</p>
<pre class="lang-java" data-nodeid="2297"><code data-language="java">find ./ | grep Spring
</code></pre>
<p data-nodeid="2298"><code data-backticks="1" data-nodeid="2576">find ./</code>递归列出当前目录下所有目录中的文件。<code data-backticks="1" data-nodeid="2578">grep</code>从<code data-backticks="1" data-nodeid="2580">find</code>的输出流中找出含有<code data-backticks="1" data-nodeid="2582">Spring</code>关键字的行。</p>
<p data-nodeid="2299">如果我们希望包含<code data-backticks="1" data-nodeid="2585">Spring</code>但不包含<code data-backticks="1" data-nodeid="2587">MyBatis</code>就可以这样操作：</p>
<pre class="lang-java" data-nodeid="2300"><code data-language="java">find ./ | grep Spring | grep -v MyBatis
</code></pre>
<p data-nodeid="2301"><code data-backticks="1" data-nodeid="2589">grep -v</code>是匹配不包含 MyBatis 的结果。</p>
<h4 data-nodeid="2302">数行数</h4>
<p data-nodeid="2303">还有一个比较常见的场景是数行数。比如你写了一个 Java 文件想知道里面有多少行，就可以使用<code data-backticks="1" data-nodeid="2593">wc -l</code>指令，如下所示：</p>
<p data-nodeid="2304"><img src="https://s0.lgstatic.com/i/image/M00/57/F5/Ciqc1F9twRqAH6ezAAD5iEQBhxE628.png" alt="Drawing 8.png" data-nodeid="2597"></p>
<p data-nodeid="2305">但是如果你想知道当前目录下有多少个文件，可以用<code data-backticks="1" data-nodeid="2599">ls | wc -l</code>，如下所示：</p>
<p data-nodeid="2306"><img src="https://s0.lgstatic.com/i/image/M00/57/F5/Ciqc1F9twSCAN0h-AABgIcsEgKI655.png" alt="Drawing 9.png" data-nodeid="2603"></p>
<p data-nodeid="2307"><strong data-nodeid="2612">接下来请你思考一个问题：我们如何知道当前</strong><code data-backticks="1" data-nodeid="2607">java</code><strong data-nodeid="2613">的项目目录下有多少行代码</strong>？</p>
<p data-nodeid="2308">提示一下。你可以使用下面这个指令：</p>
<pre class="lang-java" data-nodeid="2309"><code data-language="java">find -i <span class="hljs-string">".java"</span> ./ | wc -l
</code></pre>
<p data-nodeid="2310">快去自己动手写一写吧，你在尝试的过程中如果遇到什么问题，也可以写在留言区，我会逐一为你解答。</p>
<h4 data-nodeid="2311">中间结果</h4>
<p data-nodeid="2312">管道一个接着一个，是一个计算逻辑。有时候我们想要把中间的结果保存下来，这就需要用到<code data-backticks="1" data-nodeid="2618">tee</code>指令。<code data-backticks="1" data-nodeid="2620">tee</code>指令从标准输入流中读取数据到标准输出流。</p>
<p data-nodeid="2313">这时候，你可能会问： 老师， 这不是什么都没做吗？</p>
<p data-nodeid="2314">别急，<code data-backticks="1" data-nodeid="2624">tee</code>还有一个能力，就是自己利用这个过程把输入流中读取到的数据存到文件中。比如下面这条指令：</p>
<pre class="lang-java" data-nodeid="2315"><code data-language="java">find ./ -i <span class="hljs-string">"*.java"</span> | tee JavaList | grep Spring
</code></pre>
<p data-nodeid="2316">这句指令的意思是从当前目录中找到所有含有 Spring 关键字的 Java 文件。tee 本身不影响指令的执行，但是 tee 会把 find 指令的结果保存到 JavaList 文件中。</p>
<p data-nodeid="2317"><code data-backticks="1" data-nodeid="2627">tee</code>这个执行就像英文字母中的 T 一样，连通管道两端，下面又开了口。这个开口，在函数式编程里面叫作副作用。</p>
<h4 data-nodeid="2318">xargs</h4>
<p data-nodeid="2319">上面我们学习的内容难度，已经由小学 1 年级攀升到了小学 6 年级，最后我们来看看初中难度的<code data-backticks="1" data-nodeid="2631">xargs</code>指令。</p>
<p data-nodeid="2320"><code data-backticks="1" data-nodeid="2633">xargs</code>指令从标准数据流中构造并执行一行行的指令。<code data-backticks="1" data-nodeid="2635">xargs</code>从输入流获取字符串，然后利用空白、换行符等切割字符串，在这些字符串的基础上构造指令，最后一行行执行这些指令。</p>
<p data-nodeid="2321">举个例子，如果我们重命名当前目录下的所有 .a 的文件，想在这些文件前面加一个前缀<code data-backticks="1" data-nodeid="2638">prefix_</code>。比如说<code data-backticks="1" data-nodeid="2640">x.a</code>文件需要重命名成<code data-backticks="1" data-nodeid="2642">prefix_x.a</code>，我们就可以用<code data-backticks="1" data-nodeid="2644">xargs</code>指令构造模块化的指令。</p>
<p data-nodeid="2322">现在我们有<code data-backticks="1" data-nodeid="2647">x.a``y.a``z.a</code>三个文件，如下图所示：</p>
<p data-nodeid="2323"><img src="https://s0.lgstatic.com/i/image/M00/58/00/CgqCHl9twTWALpuzAABnixlvrS8980.png" alt="Drawing 10.png" data-nodeid="2651"></p>
<p data-nodeid="2324">然后使用下图中的指令构造我们需要的指令：</p>
<p data-nodeid="2325"><img src="https://s0.lgstatic.com/i/image/M00/58/01/CgqCHl9twT-AOUALAAE5FDR8Tiw234.png" alt="Drawing 11.png" data-nodeid="2655"></p>
<ul data-nodeid="2326">
<li data-nodeid="2327">
<p data-nodeid="2328">我们用<code data-backticks="1" data-nodeid="2657">ls</code>找到所有的文件；</p>
</li>
<li data-nodeid="2329">
<p data-nodeid="2330"><code data-backticks="1" data-nodeid="2659">-I</code>参数是查找替换符，这里我们用<code data-backticks="1" data-nodeid="2661">GG</code>替代<code data-backticks="1" data-nodeid="2663">ls</code>找到的结果；<code data-backticks="1" data-nodeid="2665">-I GG</code>后面的字符串 GG 会被替换为<code data-backticks="1" data-nodeid="2667">x.a``x.b</code>或<code data-backticks="1" data-nodeid="2669">x.z</code>；</p>
</li>
<li data-nodeid="2331">
<p data-nodeid="2332"><code data-backticks="1" data-nodeid="2671">echo</code>是一个在命令行打印字符串的指令。使用<code data-backticks="1" data-nodeid="2673">echo</code>主要是为了安全，帮助我们检查指令是否有错误。</p>
</li>
</ul>
<p data-nodeid="2333">我们用<code data-backticks="1" data-nodeid="2676">xargs</code>构造了 3 条指令。这里我再多讲一个词，叫作样板代码。如果你没有用<code data-backticks="1" data-nodeid="2678">xargs</code>指令，而是用一条条<code data-backticks="1" data-nodeid="2680">mv</code>指令去敲，这样就构成了样板代码。</p>
<p data-nodeid="2334">最后去掉 echo，就是我们想要的结果，如下所示：</p>
<p data-nodeid="2335"><img src="https://s0.lgstatic.com/i/image/M00/57/F5/Ciqc1F9twUiAOcNlAAEsaaMV4DI747.png" alt="Drawing 12.png" data-nodeid="2685"></p>
<h3 data-nodeid="2336">管道文件</h3>
<p data-nodeid="2337">上面我们花了较长的一段时间讨论匿名管道，用<code data-backticks="1" data-nodeid="2688">|</code>就可以创造和使用。匿名管道也是利用了文件系统的能力，是一种文件结构。当你学到模块六文件系统的内容，会知道匿名管道拥有一个自己的<code data-backticks="1" data-nodeid="2690">inode</code>，但不属于任何一个文件夹。</p>
<p data-nodeid="2338">还有一种管道叫作命名管道（Named Pipeline）。命名管道是要挂到文件夹中的，因此需要创建。用<code data-backticks="1" data-nodeid="2693">mkfifo</code>指令可以创建一个命名管道，下面我们来创建一个叫作<code data-backticks="1" data-nodeid="2695">pipe1</code>的命名管道，如下图所示：</p>
<p data-nodeid="2339"><img src="https://s0.lgstatic.com/i/image/M00/58/01/CgqCHl9twU-ASY8bAAC7_lc6Pr8814.png" alt="Drawing 13.png" data-nodeid="2699"></p>
<p data-nodeid="2340">命名管道和匿名管道能力类似，可以连接一个输出流到另一个输入流，也是 First In First Out。</p>
<p data-nodeid="2341">当执行<code data-backticks="1" data-nodeid="2702">cat pipe1</code>的时候，你可以观察到，当前的终端处于等待状态。因为我们<code data-backticks="1" data-nodeid="2704">cat pipe1</code>的时候<code data-backticks="1" data-nodeid="2706">pipe1</code>中没有内容。</p>
<p data-nodeid="2342">如果这个时候我们再找一个终端去写一点东西到<code data-backticks="1" data-nodeid="2709">pipe</code>中，比如说:</p>
<pre class="lang-java" data-nodeid="2343"><code data-language="java">echo <span class="hljs-string">"XXX"</span> &gt; pipe1
</code></pre>
<p data-nodeid="2344">这个时候，<code data-backticks="1" data-nodeid="2712">cat pipe1</code>就会返回，并打印出<code data-backticks="1" data-nodeid="2714">xxx</code>，如下所示：</p>
<p data-nodeid="2345"><img src="https://s0.lgstatic.com/i/image/M00/58/01/CgqCHl9twViAT-M2AADtPsSTV5c658.png" alt="Drawing 14.png" data-nodeid="2718"></p>
<p data-nodeid="2346">我们可以像上图那样演示这段程序，在<code data-backticks="1" data-nodeid="2720">cat pipe1</code>后面增加了一个<code data-backticks="1" data-nodeid="2722">&amp;</code>符号。这个<code data-backticks="1" data-nodeid="2724">&amp;</code>符号代表指令在后台执行，不会阻塞用户继续输入。然后我们通过<code data-backticks="1" data-nodeid="2726">echo</code>指令往<code data-backticks="1" data-nodeid="2728">pipe1</code>中写入东西，接着就会看到<code data-backticks="1" data-nodeid="2730">xxx</code>被打印出来。</p>
<h3 data-nodeid="2347">总结</h3>
<p data-nodeid="2348">这节课我们为了学习管道，先简单接触了进程的概念，然后学习了重定向。之后我们学习了匿名管道的应用场景，匿名管道帮助我们把 Linux 指令串联起来形成很强的计算能力。特别是<code data-backticks="1" data-nodeid="2734">xargs</code>指令支持模板化的生成指令，拓展了指令的能力。最后我们还学习了命名管道，命名管道让我们可以真实拿到一个管道文件，让多个程序之间可以方便地进行通信。</p>
<p data-nodeid="2349"><strong data-nodeid="2746">那么通过这节课的学习，你<b><strong data-nodeid="2745">现在可以</strong></b>来回答本节关联的面试题目：xargs 的作用了吗？</strong></p>
<p data-nodeid="2350">老规矩，请你先在脑海里构思下给面试官的表述，并把你的思考写在留言区，然后再来看我接下来的分析。</p>
<p data-nodeid="2351"><strong data-nodeid="2752">【解析】</strong> xargs 将标准输入流中的字符串分割成一条条子字符串，然后再按照我们自己想要的方式构建成一条条指令，大大拓展了 Linux 指令的能力。</p>
<p data-nodeid="2352">比如我们可以用来按照某种特定的方式逐个处理一个目录下所有的文件；根据一个 IP 地址列表逐个 ping 这些 IP，收集到每个 IP 地址的延迟等。</p>
<h3 data-nodeid="2353">思考题</h3>
<p data-nodeid="2354"><strong data-nodeid="2759">最后我再给你出一道高中难度的指令题目。请问下面这段 Shell 程序的作用是什么</strong>？</p>
<pre class="lang-java" data-nodeid="2355"><code data-language="java">mkfifo pipe1
mkfifo pipe2
echo -n run | cat - pipe1 &gt; pipe2 &amp;
cat &lt;pipe2 &gt; pipe1
</code></pre>
<p data-nodeid="3869" class="">你可以把你的答案、思路或者课后总结写在留言区，这样可以帮助你产生更多的思考，这也是构建知识体系的一部分。经过长期的积累，相信你会得到意想不到的收获。如果你觉得今天的内容对你有所启发，欢迎分享给身边的朋友。期待看到你的思考！</p>
<p data-nodeid="4987" class="">小编有话说：<br>
今天老师带你认识了面试中经常被问到的 Linux 指令，还初步讲解了进程的概念，但关于 Linux 的进阶知识以及最最让人头疼的网络知识，我们无法在一篇文章内讲完。想要了解更多知识，深入到真实互联网项目中学习实战技巧，<a href="https://kaiwu.lagou.com/java_basic.html?u#/index" data-nodeid="4993">也可以点击此处，即可跳转至《Java 就业急训营》介绍页面</a>。</p>

---

### 精选评论

##### 林䭽：
> 第7节了。大家学会了吗？

##### *鑫：
> 超值，老师真的很赞，通俗易懂，深入浅出。 不愧为人师

##### *豆：
> 创建了两个管道,运行上述shell 后,你在屏幕并不会看见什么,但是如果你重新打开shell ,并且输入top命令,你就会看见两个cat 命令在来回的传递命令,; 具体查看https://www.linuxjournal.com/article/2156;

##### **威：
> 为什么ls | wc -l 是统计ls命令输出的行数而 find -i "*.java" | wc -l 统计的不是find -i "*.java"输出的行数，是输出中所有文件的行数之和。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 抱歉，我给错了，试试： find ./ -iname "*.txt"  |xargs wc -l

##### **俊：
> mkfifo pipe1 # 创建pipe1 命名管道mkfifo pipe2 # 创建 pipe2 命名管道echo -n run | cat - pipe1  把标准输入和 pipe1 管道的内容后台运行输出到 pipe2 管道cat  pipe1 # 把 pipe2 的标准输入输出到 cat 和 pipe1 管道， 然后这条命令会和上面的命令形成一个循环这里有个问题就是；第四条命令 会把 第三条命令接收的 run数据分别输出到 cat 和 pipe1, 这样会不会导致 传输的 run 数据越来越多？第一次：pipe2 接收来自cat1次run, 分别向 cat 和 pipe1 输出1次 run第二次：pipe2 接收来自 cat 和 pipe1 的各1次run, 然后在分别向">cat 和 pipe1 输出2次 run第三次：pipe2 接收来自 cat 和 pipe1 的各2次run, 然后在分别向cat 和 pipe1 输出4次 run......

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 赞！

##### **队长：
> tee这个命令和peek的效果一样

##### *程：
> 主要是抽象一些重复动作，让Linux可以去实现

##### **用户2508：
> find ./ -i "*.java" | tee JavaList | grep Spring，老师我使用这条指令，Linux怎么提示find: unknown predicate `-i'

##### *兴：
> mkfifo pipe1 创建一个管道pipe1mkfifo pipe2">创建一个管道pipe2echo -n run | cat - pipe1  连接输入到pipe1,再将pipe1管道中的结果作为输入流流向pipe2.cat  pipe1 将管道二中的数据流向管道1

##### *帅：
> 老师讲的很好，深入浅出，期待更新

##### **鹏：
> find -i ".java" ./ | wc -l这个测试怎么有问题？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 抱歉，我给错了，同学试试： find ./ -iname "*.txt"  |xargs wc -l

##### *旭：
> 过去学Linux就是死记硬背操作命令，把命令用在工作流程中，但从来没理解过。唉

##### **钻：
> find -i ".java" ./ -exec cat -v {} \; | wc -l这样就可以统计到行数了。

##### **用户9719：
> 这个命令会把run字符串从pipe1  pipe2 to pipe1如此往复

##### **峰：
> 有没跟多的用法事例呢？这些都太简单有高级点的xargs用法 么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 课程对你偏简单了，你可以考虑自己再查一些资料。这个是应对非OS、运维方向高级研发岗的同学。

##### **敏：
> 请问ps加grep命令如何保留第一列列名呢?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; ps | awk 'NR==1 || /bash/'

##### **宏：
> find -i是-iname的意思吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 刚才man了一下，没找到-i这个选项，只有-iname。具体同学可以再追查下。

##### **强：
> 思考题没懂啥意思

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 同学可以看下后面的模块二的加餐内容。

