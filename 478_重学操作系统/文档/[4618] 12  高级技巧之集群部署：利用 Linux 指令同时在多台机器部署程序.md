<p data-nodeid="2361" class="">Linux 指令是由很多顶级程序员共同设计的，使用 Linux 指令解决问题的过程，就好像在体验一款优秀的产品。每次通过查资料使用 Linux 指令解决问题后，都会让我感到收获满满。在这个过程中，我不仅学会了一条指令，还从中体会到了软件设计的魅力：彼此独立，又互成一体。这就像每个 Linux 指令一样，专注、高效。回想起来，在我第一次看到管道、第一次使用 awk、第一次使用 sort，都曾有过这种感受。</p>
<p data-nodeid="2362">通过前面的学习，相信你已经掌握了一些基础指令的使用方法，今天我们继续挑战一个更复杂的问题——<strong data-nodeid="2472">用 Linux 指令管理一个集群</strong>。这属于 Linux 指令的高级技巧，所谓高级技巧并不是我们要学习更多的指令，而是要把之前所学的指令进行排列组合。当你从最初只能写几条指令、执行然后看结果，成长到具备书写一个拥有几十行、甚至上百行的 bash 脚本的能力时，就意味着你具备了解决复杂问题的能力。而最终的目标，是提升你对指令的熟练程度，锻炼工程能力。</p>
<p data-nodeid="2363">本课时，我将带你朝着这个目标努力，通过把简单的指令组合起来，分层组织成最终的多个脚本文件，解决一个复杂的工程问题：在成百上千的集群中安装一个 Java 环境。接下来，请你带着这个目标，开启今天的学习。</p>
<h3 data-nodeid="2364">第一步：搭建学习用的集群</h3>
<p data-nodeid="2365">第一步我们先搭建一个学习用的集群。这里简化一下模型。我在自己的电脑上装一个<code data-backticks="1" data-nodeid="2476">ubuntu</code>桌面版的虚拟机，然后再装两个<code data-backticks="1" data-nodeid="2478">ubuntu</code>服务器版的虚拟机。</p>
<p data-nodeid="2366">相对于桌面版，服务器版对资源的消耗会少很多。我将教学材料中桌面版的<code data-backticks="1" data-nodeid="2481">ubuntu</code>命名为<code data-backticks="1" data-nodeid="2483">u1</code>，两个用来被管理的服务器版<code data-backticks="1" data-nodeid="2485">ubuntu</code>叫作<code data-backticks="1" data-nodeid="2487">v1</code>和<code data-backticks="1" data-nodeid="2489">v2</code>。</p>
<p data-nodeid="2367">用桌面版的原因是：我喜欢<code data-backticks="1" data-nodeid="2492">ubuntu</code>漂亮的开源字体，这样会让我在给你准备素材的时候拥有一个好心情。如果你对此感兴趣，可以搜索<code data-backticks="1" data-nodeid="2494">ubuntu mono</code>，尝试把这个字体安装到自己的文本编辑器中。不过我还是觉得在<code data-backticks="1" data-nodeid="2496">ubuntu</code>中敲代码更有感觉。</p>
<p data-nodeid="2368">注意，我在这里只用了 3 台服务器，但是接下来我们要写的脚本是可以在很多台服务器之间复用的。</p>
<h3 data-nodeid="2369">第二步：循环遍历 IP 列表</h3>
<p data-nodeid="2370">你可以想象一个局域网中有很多服务器需要管理，它们彼此之间网络互通，我们通过一台主服务器对它们进行操作，即通过<code data-backticks="1" data-nodeid="2501">u1</code>操作<code data-backticks="1" data-nodeid="2503">v1</code>和<code data-backticks="1" data-nodeid="2505">v2</code>。</p>
<p data-nodeid="2371">在主服务器上我们维护一个<code data-backticks="1" data-nodeid="2508">ip</code>地址的列表，保存成一个文件，如下图所示：</p>
<p data-nodeid="2372"><img src="https://s0.lgstatic.com/i/image/M00/5E/75/CgqCHl-GsciASqucAACaCl1bXF4240.png" alt="Drawing 0.png" data-nodeid="2512"></p>
<p data-nodeid="2373">目前<code data-backticks="1" data-nodeid="2514">iplist</code>中只有两项，但是如果我们有足够的机器，可以在里面放成百上千项。接下来，请你思考<code data-backticks="1" data-nodeid="2516">shell</code>如何遍历这些<code data-backticks="1" data-nodeid="2518">ip</code>？</p>
<p data-nodeid="2374">你可以先尝试实现一个最简单的程序，从文件<code data-backticks="1" data-nodeid="2521">iplist</code>中读出这些<code data-backticks="1" data-nodeid="2523">ip</code>并尝试用<code data-backticks="1" data-nodeid="2525">for</code>循环遍历这些<code data-backticks="1" data-nodeid="2527">ip</code>，具体程序如下：</p>
<pre class="lang-shell" data-nodeid="2375"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash">!/usr/bin/bash</span>
readarray -t ips &lt; iplist
for ip in ${ips[@]}
do
&nbsp; &nbsp; echo $ip
done
</code></pre>
<p data-nodeid="2376">首行的<code data-backticks="1" data-nodeid="2530">#!</code>叫作 Shebang。Linux 的程序加载器会分析 Shebang 的内容，决定执行脚本的程序。这里我们希望用<code data-backticks="1" data-nodeid="2532">bash</code>来执行这段程序，因为我们用到的 readarray 指令是<code data-backticks="1" data-nodeid="2534">bash 4.0</code>后才增加的能力。</p>
<p data-nodeid="2377"><code data-backticks="1" data-nodeid="2536">readarray</code>指令将 iplist 文件中的每一行读取到变量<code data-backticks="1" data-nodeid="2538">ips</code>中。<code data-backticks="1" data-nodeid="2540">ips</code>是一个数组，可以用<code data-backticks="1" data-nodeid="2542">echo ${ips[@]}</code>打印其中全部的内容：<code data-backticks="1" data-nodeid="2544">@</code>代表取数组中的全部内容；<code data-backticks="1" data-nodeid="2546">$</code>符号是一个求值符号。不带<code data-backticks="1" data-nodeid="2548">$</code>的话，<code data-backticks="1" data-nodeid="2550">ips[@]</code>会被认为是一个字符串，而不是表达式。</p>
<p data-nodeid="2378"><code data-backticks="1" data-nodeid="2552">for</code>循环遍历数组中的每个<code data-backticks="1" data-nodeid="2554">ip</code>地址，<code data-backticks="1" data-nodeid="2556">echo</code>把地址打印到屏幕上。</p>
<p data-nodeid="2379">如果用<code data-backticks="1" data-nodeid="2559">shell</code>执行上面的程序会报错，因为<code data-backticks="1" data-nodeid="2561">readarray</code>是<code data-backticks="1" data-nodeid="2563">bash 4.0</code>后支持的能力，因此我们用<code data-backticks="1" data-nodeid="2565">chomd</code>为<code data-backticks="1" data-nodeid="2567">foreach.sh</code>增加执行权限，然后直接利用<code data-backticks="1" data-nodeid="2569">shebang</code>的能力用<code data-backticks="1" data-nodeid="2571">bash</code>执行，如下图所示：</p>
<p data-nodeid="2380"><img src="https://s0.lgstatic.com/i/image/M00/5E/6A/Ciqc1F-GsdSAZPtIAAF5yL5VkdQ049.png" alt="Drawing 1.png" data-nodeid="2575"></p>
<h3 data-nodeid="2381">第三步：创建集群管理账户</h3>
<p data-nodeid="2382">为了方便集群管理，通常使用统一的用户名管理集群。这个账号在所有的集群中都需要保持命名一致。比如这个集群账号的名字就叫作<code data-backticks="1" data-nodeid="2578">lagou</code>。</p>
<p data-nodeid="2383">接下来我们探索一下如何创建这个账户<code data-backticks="1" data-nodeid="2581">lagou</code>，如下图所示：</p>
<p data-nodeid="2384"><img src="https://s0.lgstatic.com/i/image/M00/5E/75/CgqCHl-GsdqAc2khAALNpLTWENc494.png" alt="Drawing 2.png" data-nodeid="2585"></p>
<p data-nodeid="2385">上面我们创建了<code data-backticks="1" data-nodeid="2587">lagou</code>账号，然后把<code data-backticks="1" data-nodeid="2589">lagou</code>加入<code data-backticks="1" data-nodeid="2591">sudo</code>分组。这样<code data-backticks="1" data-nodeid="2593">lagou</code>就有了<code data-backticks="1" data-nodeid="2595">sudo</code>成为<code data-backticks="1" data-nodeid="2597">root</code>的能力，如下图所示：</p>
<p data-nodeid="2386"><img src="https://s0.lgstatic.com/i/image/M00/5E/6A/Ciqc1F-GseCAYss5AAB9-SYXFJU693.png" alt="Drawing 3.png" data-nodeid="2601"></p>
<p data-nodeid="2387">接下来，我们设置<code data-backticks="1" data-nodeid="2603">lagou</code>用户的初始化<code data-backticks="1" data-nodeid="2605">shell</code>是<code data-backticks="1" data-nodeid="2607">bash</code>，如下图所示：</p>
<p data-nodeid="2388"><img src="https://s0.lgstatic.com/i/image/M00/5E/75/CgqCHl-GsiyAGKitAACU_gkGZRI467.png" alt="Drawing 4.png" data-nodeid="2611"></p>
<p data-nodeid="2389">这个时候如果使用命令<code data-backticks="1" data-nodeid="2613">su lagou</code>，可以切换到<code data-backticks="1" data-nodeid="2615">lagou</code>账号，但是你会发现命令行没有了颜色。因此我们可以将原来用户下面的<code data-backticks="1" data-nodeid="2617">.bashrc</code>文件拷贝到<code data-backticks="1" data-nodeid="2619">/home/lagou</code>目录下，如下图所示：</p>
<p data-nodeid="2390"><img src="https://s0.lgstatic.com/i/image/M00/5E/6A/Ciqc1F-GsjeAL_RwAAEyx32py80146.png" alt="Drawing 5.png" data-nodeid="2623"></p>
<p data-nodeid="2391">这样，我们就把一些自己平时用的设置拷贝了过去，包括终端颜色的设置。<code data-backticks="1" data-nodeid="2625">.bashrc</code>是启动<code data-backticks="1" data-nodeid="2627">bash</code>的时候会默认执行的一个脚本文件。</p>
<p data-nodeid="2392">接下来，我们编辑一下<code data-backticks="1" data-nodeid="2630">/etc/sudoers</code>文件，增加一行<code data-backticks="1" data-nodeid="2632">lagou ALL=(ALL)&nbsp; NOPASSWD:ALL</code>表示<code data-backticks="1" data-nodeid="2634">lagou</code>账号 sudo 时可以免去密码输入环节，如下图所示：</p>
<p data-nodeid="2393"><img src="https://s0.lgstatic.com/i/image/M00/5E/76/CgqCHl-Gsj6AQBXeAAEW0V065r0519.png" alt="Drawing 6.png" data-nodeid="2638"></p>
<p data-nodeid="2394">我们可以把上面的完整过程整理成指令文件，<code data-backticks="1" data-nodeid="2640">create_lagou.sh</code>：</p>
<pre class="lang-js" data-nodeid="2395"><code data-language="js">sudo useradd -m -d /home/lagou lagou
sudo passwd lagou
sudo usermod -G sudo lagou
sudo usermod --shell /bin/bash lagou
sudo cp ~<span class="hljs-regexp">/.bashrc /</span>home/lagou/
sudo chown lagou.lagou /home/lagou/.bashrc
sduo sh -c <span class="hljs-string">'echo "lagou ALL=(ALL)&nbsp; NOPASSWD:ALL"&gt;&gt;/etc/sudoers'</span>
</code></pre>
<p data-nodeid="2396">你可以删除用户<code data-backticks="1" data-nodeid="2643">lagou</code>，并清理<code data-backticks="1" data-nodeid="2645">/etc/sudoers</code>文件最后一行。用指令<code data-backticks="1" data-nodeid="2647">userdel lagou</code>删除账户，然后执行<code data-backticks="1" data-nodeid="2649">create_lagou.sh</code>重新创建回<code data-backticks="1" data-nodeid="2651">lagou</code>账户。如果发现结果一致，就代表<code data-backticks="1" data-nodeid="2653">create_lagou.sh</code>功能没有问题。</p>
<p data-nodeid="2397">最后我们想在<code data-backticks="1" data-nodeid="2656">v1``v2</code>上都执行<code data-backticks="1" data-nodeid="2658">create_logou.sh</code>这个脚本。但是你不要忘记，我们的目标是让程序在成百上千台机器上传播，因此还需要一个脚本将<code data-backticks="1" data-nodeid="2660">create_lagou.sh</code>拷贝到需要执行的机器上去。</p>
<p data-nodeid="2398">这里，可以对<code data-backticks="1" data-nodeid="2663">foreach.sh</code>稍做修改，然后分发<code data-backticks="1" data-nodeid="2665">create_lagou.sh</code>文件。</p>
<p data-nodeid="2399"><em data-nodeid="2670">foreach.sh</em></p>
<pre class="lang-js" data-nodeid="2400"><code data-language="js">#!/usr/bin/bash
readarray -t ips &lt; iplist

for ip in ${ips[@]}
do
&nbsp; &nbsp; scp ~/remote/create_lagou.sh ramroll@$ip:~/create_lagou.sh
done
</code></pre>
<p data-nodeid="2401">这里，我们在循环中用<code data-backticks="1" data-nodeid="2672">scp</code>进行文件拷贝，然后分别去每台机器上执行<code data-backticks="1" data-nodeid="2674">create_lagou.sh</code>。</p>
<p data-nodeid="2402">如果你的机器非常多，上述过程会变得非常烦琐。你可以先带着这个问题学习下面的<code data-backticks="1" data-nodeid="2677">Step 4</code>，然后再返回来重新思考这个问题，当然你也可以远程执行脚本。另外，还有一个叫作<code data-backticks="1" data-nodeid="2679">sshpass</code>的工具，可以帮你把密码传递给要远程执行的指令，如果你对这块内容感兴趣，可以自己研究下这个工具。</p>
<h3 data-nodeid="2403">第四步： 打通集群权限</h3>
<p data-nodeid="2404">接下来我们需要打通从主服务器到<code data-backticks="1" data-nodeid="2683">v1</code>和<code data-backticks="1" data-nodeid="2685">v2</code>的权限。当然也可以每次都用<code data-backticks="1" data-nodeid="2687">ssh</code>输入用户名密码的方式登录，但这并不是长久之计。 如果我们有成百上千台服务器，输入用户名密码就成为一件繁重的工作。</p>
<p data-nodeid="2405">这时候，你可以考虑利用主服务器的公钥在各个服务器间登录，避免输入密码。接下来我们聊聊具体的操作步骤：</p>
<p data-nodeid="2406">首先，需要在<code data-backticks="1" data-nodeid="2691">u1</code>上用<code data-backticks="1" data-nodeid="2693">ssh-keygen</code>生成一个公私钥对，然后把公钥写入需要管理的每一台机器的<code data-backticks="1" data-nodeid="2695">authorized_keys</code>文件中。如下图所示：我们使用<code data-backticks="1" data-nodeid="2697">ssh-keygen</code>在主服务器<code data-backticks="1" data-nodeid="2699">u1</code>中生成公私钥对。</p>
<p data-nodeid="2407"><img src="https://s0.lgstatic.com/i/image/M00/5E/76/CgqCHl-GslSAAUT5AATF-5rjGWU079.png" alt="Drawing 7.png" data-nodeid="2703"></p>
<p data-nodeid="2408">然后使用<code data-backticks="1" data-nodeid="2705">mkdir -p</code>创建<code data-backticks="1" data-nodeid="2707">~/.ssh</code>目录，<code data-backticks="1" data-nodeid="2709">-p</code>的优势是当目录不存在时，才需要创建，且不会报错。<code data-backticks="1" data-nodeid="2711">~</code>代表当前家目录。 如果文件和目录名前面带有一个<code data-backticks="1" data-nodeid="2713">.</code>，就代表该文件或目录是一个需要隐藏的文件。平时用<code data-backticks="1" data-nodeid="2715">ls</code>的时候，并不会查看到该文件，通常这种文件拥有特别的含义，比如<code data-backticks="1" data-nodeid="2717">~/.ssh</code>目录下是对<code data-backticks="1" data-nodeid="2719">ssh</code>的配置。</p>
<p data-nodeid="2409">我们用<code data-backticks="1" data-nodeid="2722">cd</code>切换到<code data-backticks="1" data-nodeid="2724">.ssh</code>目录，然后执行<code data-backticks="1" data-nodeid="2726">ssh-keygen</code>。这样会在<code data-backticks="1" data-nodeid="2728">~/.ssh</code>目录中生成两个文件，<code data-backticks="1" data-nodeid="2730">id_rsa.pub</code>公钥文件和<code data-backticks="1" data-nodeid="2732">is_rsa</code>私钥文件。 如下图所示：</p>
<p data-nodeid="2410"><img src="https://s0.lgstatic.com/i/image/M00/5E/76/CgqCHl-GsluAWyS-AAayQyKs6NY181.png" alt="Drawing 8.png" data-nodeid="2736"></p>
<p data-nodeid="2951" class="">可以看到<code data-backticks="1" data-nodeid="2953">id_rsa.pub</code>文件中是加密的字符串，我们可以把这些字符串拷贝到其他机器对应用户的<code data-backticks="1" data-nodeid="2955">~/.ssh/authorized_keys</code>文件中，当<code data-backticks="1" data-nodeid="2957">ssh</code>登录其他机器的时候，就不用重新输入密码了。 这个传播公钥的能力，可以用一个<code data-backticks="1" data-nodeid="2959">shell</code>脚本执行，这里我用<code data-backticks="1" data-nodeid="2961">transfer_key.sh</code>实现。</p>

<p data-nodeid="2412">我们修改一下<code data-backticks="1" data-nodeid="2749">foreach.sh</code>，并写一个<code data-backticks="1" data-nodeid="2751">transfer_key.sh</code>配合<code data-backticks="1" data-nodeid="2753">foreach.sh</code>的工作。<code data-backticks="1" data-nodeid="2755">transfer_key.sh</code>内容如下：</p>
<p data-nodeid="2413"><em data-nodeid="2760">foreach.sh</em></p>
<pre class="lang-js" data-nodeid="2414"><code data-language="js">#!/usr/bin/bash
readarray -t ips &lt; iplist
for ip in ${ips[@]}
do
&nbsp; &nbsp; sh ./transfer_key.sh $ip
done
</code></pre>
<p data-nodeid="2415"><em data-nodeid="2766">tranfer_key.sh</em></p>
<pre class="lang-shell" data-nodeid="2416"><code data-language="shell">ip=$1
pubkey=$(cat ~/.ssh/id_rsa.pub)
echo "execute on .. $ip"
ssh lagou@$ip "&nbsp;
mkdir -p ~/.ssh
echo $pubkey&nbsp; &gt;&gt; ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
"
</code></pre>
<p data-nodeid="2417">在<code data-backticks="1" data-nodeid="2768">foreach.sh</code>中我们执行 transfer_key.sh，并且将 IP 地址通过参数传递过去。在 transfer_key.sh 中，用<code data-backticks="1" data-nodeid="2774">$1</code>读出 IP 地址参数， 再将公钥写入变量<code data-backticks="1" data-nodeid="2776">pubkey</code>，然后登录到对应的服务器，执行多行指令。用<code data-backticks="1" data-nodeid="2778">mkdir</code>指令检查<code data-backticks="1" data-nodeid="2780">.ssh</code>目录，如不存在就创建这个目录。最后我们将公钥追加写入目标机器的<code data-backticks="1" data-nodeid="2782">~/.ssh/authorized_keys</code>中。</p>
<p data-nodeid="2418"><code data-backticks="1" data-nodeid="2784">chmod 700</code>和<code data-backticks="1" data-nodeid="2786">chmod 600</code>是因为某些特定的<code data-backticks="1" data-nodeid="2788">linux</code>版本需要<code data-backticks="1" data-nodeid="2790">.ssh</code>的目录为可读写执行，<code data-backticks="1" data-nodeid="2792">authorized_keys</code>文件的权限为只可读写。而为了保证安全性，组用户、所有用户都不可以访问这个文件。</p>
<p data-nodeid="2419">此前，我们执行<code data-backticks="1" data-nodeid="2795">foreach.sh</code>需要输入两次密码。完成上述操作后，我们再登录这两台服务器就不需要输入密码了。</p>
<p data-nodeid="2420"><img src="https://s0.lgstatic.com/i/image/M00/5E/76/CgqCHl-GsnuAC-lYAAb76OR4cFs817.png" alt="Drawing 9.png" data-nodeid="2799"></p>
<p data-nodeid="2421">接下来，我们尝试一下免密登录，如下图所示：</p>
<p data-nodeid="2422"><img src="https://s0.lgstatic.com/i/image/M00/5E/6A/Ciqc1F-GsoGANiKlAAIjYZ8fscs878.png" alt="Drawing 10.png" data-nodeid="2803"></p>
<p data-nodeid="2423">可以发现，我们登录任何一台机器，都不再需要输入用户名和密码了。</p>
<h3 data-nodeid="2424">第五步：单机安装 Java 环境</h3>
<p data-nodeid="2425">在远程部署 Java 环境之前，我们先单机完成以下 Java 环境的安装，用来收集需要执行的脚本。</p>
<p data-nodeid="2426">在<code data-backticks="1" data-nodeid="2808">ubuntu</code>上安装<code data-backticks="1" data-nodeid="2810">java</code>环境可以直接用<code data-backticks="1" data-nodeid="2812">apt</code>。</p>
<p data-nodeid="2427">我们通过下面几个步骤脚本配置 Java 环境：</p>
<pre class="lang-java" data-nodeid="2428"><code data-language="java">sudo apt install openjdk-<span class="hljs-number">11</span>-jdk
</code></pre>
<p data-nodeid="2429">经过一番等待我们已经安装好了<code data-backticks="1" data-nodeid="2816">java</code>，然后执行下面的脚本确认<code data-backticks="1" data-nodeid="2818">java</code>安装。</p>
<pre class="lang-java" data-nodeid="2430"><code data-language="java">which java
java --version
</code></pre>
<p data-nodeid="2431"><img src="https://s0.lgstatic.com/i/image/M00/5E/6B/Ciqc1F-GspCAJ0r9AAJx-kzES1k505.png" alt="Drawing 11.png" data-nodeid="2822"></p>
<p data-nodeid="2432">根据最小权限原则，执行 Java 程序我们考虑再创建一个用户<code data-backticks="1" data-nodeid="2824">ujava</code>。</p>
<pre class="lang-js" data-nodeid="2433"><code data-language="js">sudo useradd -m -d /opt/ujava ujava
sudo usermod --shell /bin/bash lagou
</code></pre>
<p data-nodeid="2434">这个用户可以不设置密码，因为我们不会真的登录到这个用户下去做任何事情。接下来我们为用户配置 Java 环境变量，如下图所示：</p>
<p data-nodeid="2435"><img src="https://s0.lgstatic.com/i/image/M00/5E/6B/Ciqc1F-GsqWAa2e2AAJosZCNXpU388.png" alt="Drawing 12.png" data-nodeid="2829"></p>
<p data-nodeid="2436">通过两次 ls 追查，可以发现<code data-backticks="1" data-nodeid="2831">java</code>可执行文件软连接到<code data-backticks="1" data-nodeid="2833">/etc/alternatives/java</code>然后再次软连接到<code data-backticks="1" data-nodeid="2835">/usr/lib/jvm/java-11-openjdk-amd64</code>下。</p>
<p data-nodeid="2437">这样我们就可以通过下面的语句设置 JAVA_HOME 环境变量了。</p>
<pre class="lang-java" data-nodeid="2438"><code data-language="java">export JAVA_HOME=/usr/lib/jvm/java-<span class="hljs-number">11</span>-openjdk-amd64/
</code></pre>
<p data-nodeid="2439">Linux 的环境变量就好比全局可见的数据，这里我们使用 export 设置<code data-backticks="1" data-nodeid="2841">JAVA_HOME</code>环境变量的指向。如果你想看所有的环境变量的指向，可以使用<code data-backticks="1" data-nodeid="2843">env</code>指令。</p>
<p data-nodeid="2440"><img src="https://s0.lgstatic.com/i/image/M00/5E/76/CgqCHl-GsrGAMIfNAAW55Kdz1xc547.png" alt="Drawing 13.png" data-nodeid="2847"></p>
<p data-nodeid="2441">其中有一个环境变量比较重要，就是<code data-backticks="1" data-nodeid="2849">PATH</code>。</p>
<p data-nodeid="2442"><img src="https://s0.lgstatic.com/i/image/M00/5E/6B/Ciqc1F-GsriACI2JAAEtgeamQNI945.png" alt="Drawing 14.png" data-nodeid="2853"></p>
<p data-nodeid="2443">如上图，我们可以使用<code data-backticks="1" data-nodeid="2855">shell</code>查看<code data-backticks="1" data-nodeid="2857">PATH</code>的值，<code data-backticks="1" data-nodeid="2859">PATH</code>中用<code data-backticks="1" data-nodeid="2861">:</code>分割，每一个目录都是<code data-backticks="1" data-nodeid="2863">linux</code>查找执行文件的目录。当用户在命令行输入一个命令，Linux 就会在<code data-backticks="1" data-nodeid="2865">PATH</code>中寻找对应的执行文件。</p>
<p data-nodeid="2444">当然我们不希望<code data-backticks="1" data-nodeid="2868">JAVA_HOME</code>配置后重启一次电脑就消失，因此可以把这个环境变量加入<code data-backticks="1" data-nodeid="2870">ujava</code>用户的<code data-backticks="1" data-nodeid="2872">profile</code>中。这样只要发生用户登录，就有这个环境变量。</p>
<pre class="lang-java" data-nodeid="2445"><code data-language="java">sudo sh -c <span class="hljs-string">'echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/" &gt;&gt; /opt/ujava/.bash_profile'</span>
</code></pre>
<p data-nodeid="2446">将<code data-backticks="1" data-nodeid="2875">JAVA_HOME</code>加入<code data-backticks="1" data-nodeid="2877">bash_profile</code>，这样后续远程执行<code data-backticks="1" data-nodeid="2879">java</code>指令时就可以使用<code data-backticks="1" data-nodeid="2881">JAVA_HOME</code>环境变量了。</p>
<p data-nodeid="2447">最后，我们将上面所有的指令整理起来，形成一个<code data-backticks="1" data-nodeid="2884">install_java.sh</code>。</p>
<pre class="lang-java" data-nodeid="2448"><code data-language="java">sudo apt -y install openjdk-<span class="hljs-number">11</span>-jdk
sudo useradd -m -d /opt/ujava ujava
sudo usermod --shell /bin/bash ujava
sudo sh -c <span class="hljs-string">'echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/" &gt;&gt; /opt/ujava/.bash_profile'</span>
</code></pre>
<p data-nodeid="2449"><code data-backticks="1" data-nodeid="2886">apt</code>后面增了一个<code data-backticks="1" data-nodeid="2888">-y</code>是为了让执行过程不弹出确认提示。</p>
<h3 data-nodeid="2450">第六步：远程安装 Java 环境</h3>
<p data-nodeid="2451">终于到了远程安装 Java 环境这一步，我们又需要用到<code data-backticks="1" data-nodeid="2892">foreach.sh</code>。为了避免每次修改，你可以考虑允许<code data-backticks="1" data-nodeid="2894">foreach.sh</code>带一个文件参数，指定需要远程执行的脚本。</p>
<p data-nodeid="2452"><em data-nodeid="2900"><strong data-nodeid="2899">foreach.sh</strong></em></p>
<pre class="lang-js" data-nodeid="2453"><code data-language="js">#!/usr/bin/bash
readarray -t ips &lt; iplist

script=$1
for ip in ${ips[@]}
do
&nbsp; &nbsp; ssh $ip 'bash -s' &lt; $script
done
</code></pre>
<p data-nodeid="2454">改写后的<code data-backticks="1" data-nodeid="2902">foreach</code>会读取第一个执行参数作为远程执行的脚本文件。 而<code data-backticks="1" data-nodeid="2904">bash -s</code>会提示使用标准输入流作为命令的输入；<code data-backticks="1" data-nodeid="2906">&lt; $script</code>负责将脚本文件内容重定向到远程<code data-backticks="1" data-nodeid="2908">bash</code>的标准输入流。</p>
<p data-nodeid="2455">然后我们执行<code data-backticks="1" data-nodeid="2911">foreach.sh install_java.sh</code>，机器等待 1 分钟左右，在执行结束后，可以用下面这个脚本检测两个机器中的安装情况。</p>
<p data-nodeid="2456"><em data-nodeid="2917"><strong data-nodeid="2916">check.sh</strong></em></p>
<pre class="lang-java" data-nodeid="2457"><code data-language="java">sudo -u ujava -i /bin/bash -c <span class="hljs-string">'echo $JAVA_HOME'</span>
sudo -u ujava -i java --version
</code></pre>
<p data-nodeid="2458"><code data-backticks="1" data-nodeid="2918">check.sh</code>中我们切换到<code data-backticks="1" data-nodeid="2920">ujava</code>用户去检查<code data-backticks="1" data-nodeid="2922">JAVA_HOME</code>环境变量和 Java 版本。执行的结果如下图所示：</p>
<p data-nodeid="2459"><img src="https://s0.lgstatic.com/i/image/M00/5E/76/CgqCHl-GstWAFW9yAAQXx_nh6dw719.png" alt="Drawing 15.png" data-nodeid="2926"></p>
<h3 data-nodeid="2460">总结</h3>
<p data-nodeid="2461">这节课我们所讲的场景是自动化运维的一些皮毛。通过这样的场景练习，我们复习了很多之前学过的 Linux 指令。在尝试用脚本文件构建一个又一个小工具的过程中，可以发现复用很重要。</p>
<p data-nodeid="2462">在工作中，优秀的工程师，总是善于积累和复用，而<code data-backticks="1" data-nodeid="2930">shell</code>脚本就是积累和复用的利器。如果你第一次安装<code data-backticks="1" data-nodeid="2932">java</code>环境，可以把今天的安装脚本保存在自己的笔记本中，下次再安装就能自动化完成了。除了积累和总结，另一个非常重要的就是你要尝试自己去查资料，包括使用<code data-backticks="1" data-nodeid="2934">man</code>工具熟悉各种指令的使用方法，用搜索引擎查阅资料等。</p>
<h3 data-nodeid="2463">课后练习题</h3>
<p data-nodeid="2464"><strong data-nodeid="2949">最后我再给你出一道需要查阅资料的题目：~/.bashrc ~/.bash_profile, ~/.profile 和 /etc/profile 的区别是什么</strong>？</p>
<p data-nodeid="2465" class="">你可以把你的答案、思路或者课后总结写在留言区，这样可以帮助你产生更多的思考，这也是构建知识体系的一部分。经过长期的积累，相信你会得到意想不到的收获。如果你觉得今天的内容对你有所启发，欢迎分享给身边的朋友。期待看到你的思考！</p>

---

### 精选评论

##### Zayn：
> 虽然这里大部分知识在我的摸索过程中都已经学习到了，但是如果之前能看到这样简明扼要的文章，真的能少走很多弯路。

##### **康：
> Login shell 会执行 *profile 文件，但是不会自动执行 rc 文件。通常防止环境变量。作为 Login shell">子进程的交互式 shell 会执行 ~/.bashrc，但是不会执行 *profile 文件。https://www.linuxjournal.com/content/profiles-and-rc-files 可以看这个链接。小知识：rc 是 run command 的缩写。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 赞！

##### **华：
> 听到后面吃力了，还要多看点不同的资料

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 是滴呀，打好基础非常重要，加油！！

##### *辉：
> 可以看到id_rsa.pub文件中是加密的字符串，我们可以把这些字符串拷贝到其他机器对应用户的~/.ssh/known_hosts文件中这里写错了，根据上下文，这里要写入的是authorized_keys文件

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 感谢同学的反馈，这里是林䭽老师笔误，确实写错了。我们已经修正了。

##### **云：
> /etc/profile：">系统所有用户设置环境信息~/.bash_profile, ~/.profile：为当前用户设置环境信息~/.bashrc：为当前用户设置bash shell的bash信息

##### *贝：
> 使用expect的话，可以在分发公钥的时候自动输入密码，不过这种方案的缺点就是密码会写成明文。然后，复制公钥可以用ssh-copy-id命令完成。

##### Zayn：
> 复制公钥可以用ssh-copy-id命令

##### *尚：
> Rsync 更强大

##### **8109：
> 模块二就是linux系统基础知识学习

