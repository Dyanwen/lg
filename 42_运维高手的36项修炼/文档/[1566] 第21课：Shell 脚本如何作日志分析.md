<p>上个课时我们讲解了日志分析的一些 Linux 命令，如果想对一些偏大型或复杂场景进行分析，而又缺乏系统的日志收集检索系统，此时就需要借助脚本（Shell、Python、PhP 等）来帮助我们进行日志分析，本课时我们就讲解如何通过 Shell 来进行日志分析，并介绍一些比较高效的方法。</p>
<h4>grep 命令</h4>
<p>首先为你讲解一个常用命令 grep ， grep 是 Linux 常用对日志文件进行筛选查找的命令。它的常用使用方式是这样的：</p>
<pre><code data-language="SQL" class="lang-SQL">grep [选项] PATTERN filename
</code></pre>
<p>grep 后面加具体的执行选项，然后加上要查找的内容（PATTERN），PATTERN 可以是模式匹配的关键文字内容或正则表达式，filename 就是查找的目标文件名称。</p>
<p>这是 grep 的一种典型使用方式，除此之处 grep 也支持使用（command1|grep [选项] PATTERN）使用方式，这个在前面的课时我们多次应用到。</p>
<p>接下来我们主要讲一讲，grep 对文件的查找都有哪些具体的选项？</p>
<p>-i，表示可以忽略大小写，也就是可以忽略查找的文字的大小写，不管大写还是小写，只要匹配这个字符串的文字都可以被筛选出来。</p>
<p>-l（小写字母 l），它表示结果中只显示含有匹配内容的文件名称或路径。</p>
<p>-R，递归查询，查找在某一级目录下，有多少个文件内容里含有指定的内容字符串时，它可以在一个目录层级下发起递归性查询。</p>
<p>-o，表示结果中打印匹配出的内容。</p>
<p>-B，表示查找在匹配关键字符串之前，有多少行内容。</p>
<p>-A，表示匹配关键字的这一行之后有多少行内容。如果我们把 -B 和 -A 都同时加上，比如 -B2 和 -A2，就会把匹配关键字的前 2 行和后 2 行内容整体打印显示出来。</p>
<p>以上这就是 grep 里的一些常见选项，我们来看一个 grep 的正则匹配，需要使用 grep -E 或 egrep 命令，可以直接进行正则的匹配。</p>
<p>这里有一个匹配正则表达式的样例：</p>
<pre><code data-language="java" class="lang-java">egrep ‘[^<span class="hljs-number">0</span>][<span class="hljs-number">0</span>-<span class="hljs-number">9</span>]{<span class="hljs-number">0</span>,<span class="hljs-number">2</span>}\.[<span class="hljs-number">0</span>-<span class="hljs-number">9</span>]{<span class="hljs-number">1</span>,<span class="hljs-number">3</span>}\.[<span class="hljs-number">0</span>-<span class="hljs-number">9</span>]{<span class="hljs-number">1</span>,<span class="hljs-number">3</span>}\.[<span class="hljs-number">0</span>-<span class="hljs-number">9</span>]{<span class="hljs-number">1</span>,<span class="hljs-number">3</span>}’ ./test -c
</code></pre>
<p>如上的 egrep 命令，单引号里是具体的正则表达式（[^0][0-9]{0,2}.[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}），‘./test’ 表示需要查找的目标文件，接下来我们分析正则表达式是什么作用。</p>
<p>从左到右分析，首先拆开分析这样的一段[^0][0-9]{0,2}\，第一个方括号中的 ^0，它表示匹配第一个数值是非零开头的任意数字，后面是 [0-9]，表示第 2 个数开始，数值需要 0~9 之间，而 {0,2} 表示限制了第二个数字开始后面数字次数（可以不出现、或出现第二数、或出现到第三个数） ，它表述允许[0-9]数值出现的范围。</p>
<p>通过一个小数点分割后，是这样的一段 [0-9]{1,3}\，匹配原理也是一样。</p>
<p>通过整个这样的一串表达式，查找 test 文件里是否有 IP 地址，并把匹配 IP 地址的行信息全部都打印出来。</p>
<h4>awk 命令</h4>
<p>第 2 个介绍命令就是 awk，它是一个非常强大的文件分析工具和编程语言，可以逐行扫描文件，从第一行到最后一行寻找匹配的特定模式的行，并在这些行上进行想要的操作。</p>
<p>具体的使用方式是这样的：</p>
<pre><code data-language="java" class="lang-java">awk ‘pattern’ filename
</code></pre>
<p>awk 后面加想要匹配的文件内容，然后再加这个文件的路径或者名称。</p>
<p>示例：awk -F: ‘/root/’ /etc/passwd</p>
<p>如上有一个使用事例，awk -F:，它表示查找 /etc 下的 passwd 文件，这里会以冒号进行内容上按列分割，然后前面加了一个匹配的内容（pattern），/root/ 表示是查找 /etc/ passwd 文件内容是否包含 root 这个关键字，然后打印对应的行打印。</p>
<p>第 2 个使用方式是：</p>
<pre><code data-language="java" class="lang-java">awk ‘{action}’ filename
</code></pre>
<p>示例：awk -F: ‘{print $1}’ /etc/passwd</p>
<p>我们同样看一下这个示例，这里的 {print $1}，$1 表示打印 /etc/passwd 文件中每行的第 1 列（及系统上的用户名）。</p>
<p>第3种使用方式是：</p>
<pre><code data-language="java" class="lang-java">awk ‘pattern {action}’ filename
</code></pre>
<p>我们看到它既做了查找，后面又做了 action。这个方式会更加全面。这里同样有一个例子：</p>
<pre><code data-language="java" class="lang-java">awk -F: ‘/root/{print $<span class="hljs-number">1</span>,$<span class="hljs-number">3</span>}’ /etc/passwd
</code></pre>
<p>首先它会做一个查找，匹配有 root 关键字的这一行，然后打印它的第 1 列和第 3 列，这里 awk 在这里既做关键词的匹配，也做了 action({print $1,$3})。</p>
<p>经验来看，awk 的强大主要体现在它的 action （动作指令），它可以对文件内容进行丰富灵活的处理，关于action的使用格式可以细分如下：</p>
<pre><code data-language="java" class="lang-java">格式<span class="hljs-number">1</span>:BEGIN{} {} END{}
</code></pre>
<p>BEGIN{} 表示在 awk 处理行前，需要执行的动作。中间的 {} 表示在 awk 执行过程中，所需要执行的动作，END{} 表示处理完所有内容以后的动作。</p>
<p>值得注意的是：实际使用 action 这种格式中，BEGIN{} 和 END{} 我们可以选择不加。</p>
<p>我们来看一下这个案例：</p>
<pre><code data-language="java" class="lang-java">awk -F: <span class="hljs-string">'length($1)==5{count++;print $1}END{print "count is: "count}'</span> /etc/passwd
</code></pre>
<p>它只加了中间括号 {} 和 END{}，在这个样例的 awk 里面，它对 passwd 文件做了一个操作，用来匹配 /etc/passwd 里每一行第 1 列的内容（Linux 系统用户名）长度是否等于 5，如果是等于 5，则说明匹配到要求的条件（length($1)==5）。条件成立以后在处理过程中会做一个计数（count++），并把第 1 列的内容打印出来，执行完 awk 以后，再把 count 的结果打印出来，我们就知道总体的匹配有多少个。</p>
<p>这就是 awk 的技术样例。那么再来看一下这样的一个样例：</p>
<pre><code data-language="java" class="lang-java">Cat xxx.log|awk 'BEGIN{max=0;min=1} 
{if ($4 ~ /jeson/ &amp;&amp; $8==200){ 
sum+=$NF; count+=1; 
if($9 &gt; max) max=$9 fi;if($9 &lt; min) min=$9 fi; 
}
}
END {print "Average = ", sum/count;print "Max = ", max;print "Min", min}'
</code></pre>
<p>我们上一课时的内容里面讲到过进行性能分析时，通常需要对 Nginx 的 access 日志进行响应时间的分析，如果我们想了解某一个接口它总体的情况，如最大响应时间是多久，最小的响应时间是多久，或者它的平均响应时间是多久，这时就可能需要通过 awk 来对日志进行整体分析，这里来看一下。</p>
<p>补充：xxx.log的日志格式：</p>
<pre><code data-language="java" class="lang-java">log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      ‘“$http_user_agent” “$http_x_forwarded_for” $upstream_response_time';
</code></pre>
<p>这里的使用方式是这样的：</p>
<p>首先 cat 一个日志，把日志整打印出来，然后通过管道符给 awk 处理，这里就用到了 BEGIN{} 这个 action 行为，然后还有中间的括号行为，以及 END{}。</p>
<p>接下来 BEGIN action(max=0；min=1) 表示的是在执行 awk 之前所需要进行预处理的部分，这里先设置变量 max 等于 0，还有一个最小的值 min 等于 1。在进行处理中的括号里会有一个整体的语句进行判断，判断 $4（也就是 Cat xxx.log 这个文件的第 4 列是否匹配关键字 jeson），同时还要满足另外一个条件 $8==200（也就是 Nginx log 里面的返回状态码是否满足 200）。如果这两个条件都成立，这个时候就会用 sum 进行计数，$NF 表示日志里面最后一列的内容，sum+=$NF 表示 upstream_response_time 的数值进行累加，得到整体 upstream_response_time 的时间，也就是将 Nginx 转发到后端所有请求响应时间求和，赋值给 sum 变量。count+=1 表示 count 同时做一个计数器来自增。</p>
<p>if($9 &gt; max) max=$NF fi;if($NF &lt; min) min=$NF ，接下来会做一个判断，判断 $NF（也就是最后的响应时间）是否大于 max（第一次是默认值 0），如果大于 0 的话，那么它会把 max 的值重新替换。同样，也会和最小值进行比较，如果它小于最小值，就又会把最小变量 min 做一个替换，从而循环的去进行判断，通过每一个请求拿到整体最大请求和最小请求的时间，并且都赋值到对应的变量里去，分别是 count、sum、max 和 min。</p>
<p>END {print "Average = ", sum/count;print "Max = ", max;print "Min", min}'<br>
最后再整体把它的结果输出，这里就用到了 END 后面的括号了。它会把这个文件内容里面的平均响应时间求出来，通过 sum 变量，用整体的响应时间除以请求个数（sum/count），这样就得到了一个平均的响应时间。然后把 max 变量打印出来，得到了最大响应时间。min 这个时变量打印出来就是最小打印时间。</p>
<h4>ag 命令</h4>
<p>接下来还要为你介绍一个文件分析命令 ag，ag 命令相比 awk 和 grep 命令来我们感觉会更加新鲜，它的优势是性能比 grep 和 awk 做文件查找会更高效，并它默认支持正则方式，而不用通过 egrep 这种在里面加一些 -e 的选项，可以直接进行正则匹配。</p>
<p>ag 使用选项，跟 grep 命令基本上都是一致的。我们可以看一下：</p>
<ul>
<li>-A 表示查找指定行的后多少行。</li>
<li>-B 表示查找指定行的前面有多少行。</li>
<li>-context 表示匹配特定行的前后多少行。</li>
</ul>
<p>等等，基本上和 grep 的命令选项的功能是几乎一致的。</p>
<p>接下来我们演示一个 Shell 作日志分析的脚本，它用到了 ag 命令的使用方式，Shell 脚本里，大部分对于文件关键字的查找都是基于 ag 命令来实现的。</p>
<p>这个Shell 脚本主要可以实现这样的一些功能：</p>
<ol>
<li>统计 Top 20 地址</li>
<li>SQL 注入分析</li>
<li>SQL 注入 FROM 查询统计</li>
<li>扫描器/常用黑客工具</li>
<li>漏洞利用检测</li>
<li>敏感路径访问</li>
<li>文件包含攻击</li>
<li>Web Shell</li>
<li>寻找响应长度的 URL Top 20</li>
<li>寻找罕见的脚本文件访问</li>
<li>寻找 302 跳转的脚本文件</li>
</ol>
<p>整体 Nginx 的日志进行分析，接下来登录到一台测试机上面，然后 cd t21 目录下，有一个 nginx_check.sh 的 Shell 脚本。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/17/C3/Ciqah16oCk2AEbinAAF6FZc0C8Y862.png" alt="1.png"></p>
<p>我们首先来介绍一下 Shell 脚本的整体结构，最上面的这一段语句是做一个文件路径（/tmp/logs）的判断，如果存在的话，就会先把之前的文件做一个清空。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/17/C3/Ciqah16oClaAAuEjAAHN0RjwOLs251.png" alt="2.png"></p>
<p>这是因为执行日志分析的脚本时，会把每一项的功能得到的统计结果，输出到 tmp 路径下面的 logs 目录下，并且以对应的文件名进行归纳。当我们执行完脚本以后，想要了解每一项结果内容的时候，就可以到该路径下面去查看对应的结果。</p>
<p>这里需要填写 Nginx 的 access 日志目录，用于分析的文件 access 日志路径。接下来做日志路径判断，如果日志路径不存在，那么脚本执行就会中断退出。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/0A/94/CgoCgV6oCnSAMMVaAAHrrrcMxQM494.png" alt="3.png"></p>
<p>再往下看的话，这里就是做系统版本检测，这个脚本需要在 Debian 操作系统或者 Ubuntu 、Centos 这样的操作系统上，如果操作系统不能满足的话也会退出。</p>
<p>接下来就要用到我们刚刚讲到的 ag 命令。在没有 ag 命令情况下，脚本会先提示并进行安装，对应的使用 Yu m 或者是 apt get 包管理器安装。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/17/C3/Ciqah16oCnyAU7F5AAK3m1tcm1U889.png" alt="4.png"></p>
<p>下面就具体每一项的功能分析：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/0A/94/CgoCgV6oCoSABFmQAADl0mRWcSA337.png" alt="5.png"></p>
<p>这里分析的就是访问 Top20 的 IP 地址，那么 ag 命令就做一个正则匹配，把所有的地址打印出来，然后通过 sort 来进行排序，uniq 来进行统计，然后再得到由大到小的结果，并且过滤出前 20 行数，然后同时把内容给到 tee 这个命令，它会把输出内容重定向到日志结果目录（/tmp/logs/top20.log），整体执行完毕后，如果我们想要看这一项分析结果的话，那么就可以在 log 目录下 Top20.log 文件里查看。另外 tee 命令还可以支持在终端输出，也就是既把结果放在终端输出。</p>
<p>所以这样就完成了第 1 项的日志功能分析，分析出访次数问前 20 的 IP 地址，并且打印。后面的每一项功能分析其实都是类似的原理，通过 ag 命令来对文件进行关键字查找，并且进行分析和统计。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/0A/94/CgoCgV6oCouACYpDAAOHWLR8AZM738.png" alt="6.png"></p>
<p>好了，最后执行 sh nginx_check.sh，就可以开始 nginx access 日志来分析，我们在控制台终端关注它的执行过程和进度，并且在结果目录中详细分析日志得到的结果内容。</p>

---

### 精选评论


