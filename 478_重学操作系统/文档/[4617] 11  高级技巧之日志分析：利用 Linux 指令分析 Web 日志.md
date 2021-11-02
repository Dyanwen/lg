<p data-nodeid="55388">著名的黑客、自由软件运动的先驱理查德.斯托曼说过，“编程不是科学，编程是手艺”。可见，要想真正搞好编程，除了学习理论知识，还需要在实际的工作场景中进行反复的锤炼。</p>



<p data-nodeid="54482">所以今天我们将结合实际的工作场景，带你利用 Linux 指令分析 Web 日志，这其中包含很多小技巧，掌握了本课时的内容，将对你将来分析线上日志、了解用户行为和查找问题有非常大地帮助。</p>
<p data-nodeid="54483">本课时将用到一个大概有 5W 多条记录的<code data-backticks="1" data-nodeid="54561">nginx</code>日志文件，你可以在<a href="https://github.com/ramroll/lagou-os/blob/main/access.log" data-nodeid="54565"> GitHub</a>上下载。 下面就请你和我一起，通过分析这个<code data-backticks="1" data-nodeid="54567">nginx</code>日志文件，去锤炼我们的手艺。</p>
<h3 data-nodeid="54484">第一步：能不能这样做？</h3>
<p data-nodeid="56578">当我们想要分析一个线上文件的时候，首先要思考，能不能这样做？ 这里你可以先用<code data-backticks="1" data-nodeid="56581">htop</code>指令看一下当前的负载。如果你的机器上没有<code data-backticks="1" data-nodeid="56583">htop</code>，可以考虑用<code data-backticks="1" data-nodeid="56585">yum</code>或者<code data-backticks="1" data-nodeid="56587">apt</code>去安装。</p>
<p data-nodeid="56579" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/7F/CgqCHl-BkJ6AcP32AAduMy8fcSw412.png" alt="Drawing 0.png" data-nodeid="56591"></p>


<p data-nodeid="54487">如上图所示，我的机器上 8 个 CPU 都是 0 负载，<code data-backticks="1" data-nodeid="54583">2G</code>的内存用了一半多，还有富余。 我们用<code data-backticks="1" data-nodeid="54585">wget</code>将目标文件下载到本地（如果你没有 wget，可以用<code data-backticks="1" data-nodeid="54587">yum</code>或者<code data-backticks="1" data-nodeid="54589">apt</code>安装）。</p>
<pre class="lang-java" data-nodeid="54488"><code data-language="java">wget 某网址（自己替代）
</code></pre>
<p data-nodeid="57780">然后我们用<code data-backticks="1" data-nodeid="57783">ls</code>查看文件大小。发现这只是一个 7M 的文件，因此对线上的影响可以忽略不计。如果文件太大，建议你用<code data-backticks="1" data-nodeid="57785">scp</code>指令将文件拷贝到闲置服务器再分析。下图中我使用了<code data-backticks="1" data-nodeid="57787">--block-size</code>让<code data-backticks="1" data-nodeid="57789">ls</code>以<code data-backticks="1" data-nodeid="57791">M</code>为单位显示文件大小。</p>
<p data-nodeid="57781" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/74/Ciqc1F-BkKeAQDs9AACqJbZ2jCM025.png" alt="Drawing 1.png" data-nodeid="57795"></p>


<p data-nodeid="54491">确定了当前机器的<code data-backticks="1" data-nodeid="54606">CPU</code>和内存允许我进行分析后，我们就可以开始第二步操作了。</p>
<h3 data-nodeid="54492">第二步：LESS 日志文件</h3>
<p data-nodeid="58984">在分析日志前，给你提个醒，记得要<code data-backticks="1" data-nodeid="58987">less</code>一下，看看日志里面的内容。之前我们说过，尽量使用<code data-backticks="1" data-nodeid="58989">less</code>这种不需要读取全部文件的指令，因为在线上执行<code data-backticks="1" data-nodeid="58991">cat</code>是一件非常危险的事情，这可能导致线上服务器资源不足。</p>
<p data-nodeid="58985" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/7F/CgqCHl-BkK6AcDGvAAjaPXe-Nbc605.png" alt="Drawing 2.png" data-nodeid="58995"></p>


<p data-nodeid="54495">如上图所示，我们看到<code data-backticks="1" data-nodeid="54620">nginx</code>的<code data-backticks="1" data-nodeid="54622">access_log</code>每一行都是一次用户的访问，从左到右依次是：</p>
<ul data-nodeid="54496">
<li data-nodeid="54497">
<p data-nodeid="54498">IP 地址；</p>
</li>
<li data-nodeid="54499">
<p data-nodeid="54500">时间；</p>
</li>
<li data-nodeid="54501">
<p data-nodeid="54502">HTTP 请求的方法、路径和协议版本、返回的状态码；</p>
</li>
<li data-nodeid="54503">
<p data-nodeid="54504">User Agent。</p>
</li>
</ul>
<h3 data-nodeid="54505">第三步：PV 分析</h3>
<p data-nodeid="61388">PV（Page View），用户每访问一个页面就是一次<code data-backticks="1" data-nodeid="61391">Page View</code>。对于<code data-backticks="1" data-nodeid="61393">nginx</code>的<code data-backticks="1" data-nodeid="61395">acess_log</code>来说，分析 PV 非常简单，我们直接使用<code data-backticks="1" data-nodeid="61397">wc -l</code>就可以看到整体的<code data-backticks="1" data-nodeid="61399">PV</code>。</p>
<p data-nodeid="61389" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/74/Ciqc1F-BkL6AGiY-AABQPMnGu40979.png" alt="Drawing 3.png" data-nodeid="61403"></p>




<p data-nodeid="54508">如上图所示：我们看到了一共有 51462 条 PV。</p>
<h3 data-nodeid="54509">第四步：PV 分组</h3>
<p data-nodeid="54510">通常一个日志中可能有几天的 PV，为了得到更加直观的数据，有时候需要按天进行分组。为了简化这个问题，我们先来看看日志中都有哪些天的日志。</p>
<p data-nodeid="54511">使用<code data-backticks="1" data-nodeid="54647">awk '{print $4}' access.log&nbsp; | less</code>可以看到如下结果。<code data-backticks="1" data-nodeid="54649">awk</code>是一个处理文本的领域专有语言。这里就牵扯到领域专有语言这个概念，英文是Domain Specific Language。领域专有语言，就是为了处理某个领域专门设计的语言。比如awk是用来分析处理文本的DSL，html是专门用来描述网页的DSL，SQL是专门用来查询数据的DSL……大家还可以根据自己的业务设计某种针对业务的DSL。</p>
<p data-nodeid="62592">你可以看到我们用<code data-backticks="1" data-nodeid="62595">$4</code>代表文本的第 4 列，也就是时间所在的这一列，如下图所示：</p>
<p data-nodeid="62593" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/7F/CgqCHl-BkMaAb421AAGUr-N08hM187.png" alt="Drawing 4.png" data-nodeid="62599"></p>


<p data-nodeid="63788">我们想要按天统计，可以利用 <code data-backticks="1" data-nodeid="63791">awk</code>提供的字符串截取的能力。</p>
<p data-nodeid="63789" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/7F/CgqCHl-BkMuAKo9UAAIcPR902XQ858.png" alt="Drawing 5.png" data-nodeid="63795"></p>


<p data-nodeid="54516">上图中，我们使用<code data-backticks="1" data-nodeid="54664">awk</code>的<code data-backticks="1" data-nodeid="54666">substr</code>函数，数字<code data-backticks="1" data-nodeid="54668">2</code>代表从第 2 个字符开始，数字<code data-backticks="1" data-nodeid="54670">11</code>代表截取 11 个字符。</p>
<p data-nodeid="64984">接下来我们就可以分组统计每天的日志条数了。</p>
<p data-nodeid="64985" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/7F/CgqCHl-BkNGAB-VgAASNmct9nQA628.png" alt="Drawing 6.png" data-nodeid="64989"></p>


<p data-nodeid="54519">上图中，使用<code data-backticks="1" data-nodeid="54677">sort</code>进行排序，然后使用<code data-backticks="1" data-nodeid="54679">uniq -c</code>进行统计。你可以看到从 2015 年 5 月 17 号一直到 6 月 4 号的日志，还可以看到每天的 PV 量大概是在 2000~3000 之间。</p>
<h3 data-nodeid="54520">第五步：分析 UV</h3>
<p data-nodeid="66178">接下来我们分析 UV。UV（Uniq Visitor），也就是统计访问人数。通常确定用户的身份是一个复杂的事情，但是我们可以用 IP 访问来近似统计 UV。</p>
<p data-nodeid="66179" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/74/Ciqc1F-BkNeAam2YAACxCjlKsvc488.png" alt="Drawing 7.png" data-nodeid="66183"></p>


<p data-nodeid="54523">上图中，我们使用 awk 去打印<code data-backticks="1" data-nodeid="54689">$1</code>也就是第一列，接着<code data-backticks="1" data-nodeid="54691">sort</code>排序，然后用<code data-backticks="1" data-nodeid="54693">uniq</code>去重，最后用<code data-backticks="1" data-nodeid="54695">wc -l</code>查看条数。 这样我们就知道日志文件中一共有<code data-backticks="1" data-nodeid="54697">2660</code>个 IP，也就是<code data-backticks="1" data-nodeid="54699">2660</code>个 UV。</p>
<h3 data-nodeid="54524">第六步：分组分析 UV</h3>
<p data-nodeid="54525">接下来我们尝试按天分组分析每天的 UV 情况。这个情况比较复杂，需要较多的指令，我们先创建一个叫作<code data-backticks="1" data-nodeid="54703">sum.sh</code>的<code data-backticks="1" data-nodeid="54705">bash</code>脚本文件，写入如下内容：</p>
<pre class="lang-shell" data-nodeid="54526"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash">!/usr/bin/bash</span>
awk '{print substr($4, 2, 11) " " $1}' access.log |\
	sort | uniq |\
	awk '{uv[$1]++;next}END{for (ip in uv) print ip, uv[ip]}'
</code></pre>
<p data-nodeid="54527">具体分析如下。</p>
<ul data-nodeid="54528">
<li data-nodeid="54529">
<p data-nodeid="54530">文件首部我们使用<code data-backticks="1" data-nodeid="54709">#!</code>，表示我们将使用后面的<code data-backticks="1" data-nodeid="54711">/usr/bin/bash</code>执行这个文件。</p>
</li>
<li data-nodeid="54531">
<p data-nodeid="54532">第一次<code data-backticks="1" data-nodeid="54714">awk</code>我们将第 4 列的日期和第 1 列的<code data-backticks="1" data-nodeid="54716">ip</code>地址拼接在一起。</p>
</li>
<li data-nodeid="54533">
<p data-nodeid="54534">下面的<code data-backticks="1" data-nodeid="54719">sort</code>是把整个文件进行一次字典序排序，相当于先根据日期排序，再根据 IP 排序。</p>
</li>
<li data-nodeid="54535">
<p data-nodeid="54536">接下来我们用<code data-backticks="1" data-nodeid="54722">uniq</code>去重，日期 +IP 相同的行就只保留一个。</p>
</li>
<li data-nodeid="54537">
<p data-nodeid="54538">最后的<code data-backticks="1" data-nodeid="54725">awk</code>我们再根据第 1 列的时间和第 2 列的 IP 进行统计。</p>
</li>
</ul>
<p data-nodeid="54539">为了理解最后这一行描述，我们先来简单了解下<code data-backticks="1" data-nodeid="54728">awk</code>的原理。</p>
<p data-nodeid="54540"><code data-backticks="1" data-nodeid="54730">awk</code>本身是逐行进行处理的。因此我们的<code data-backticks="1" data-nodeid="54732">next</code>关键字是提醒<code data-backticks="1" data-nodeid="54734">awk</code>跳转到下一行输入。 对每一行输入，<code data-backticks="1" data-nodeid="54736">awk</code>会根据第 1 列的字符串（也就是日期）进行累加。之后的<code data-backticks="1" data-nodeid="54738">END</code>关键字代表一个触发器，就是 END 后面用 {} 括起来的语句会在所有输入都处理完之后执行——当所有输入都执行完，结果被累加到<code data-backticks="1" data-nodeid="54740">uv</code>中后，通过<code data-backticks="1" data-nodeid="54742">foreach</code>遍历<code data-backticks="1" data-nodeid="54744">uv</code>中所有的<code data-backticks="1" data-nodeid="54746">key</code>，去打印<code data-backticks="1" data-nodeid="54748">ip</code>和<code data-backticks="1" data-nodeid="54750">ip</code>对应的数量。</p>
<p data-nodeid="67372">编写完上面的脚本之后，我们保存退出编辑器。接着执行<code data-backticks="1" data-nodeid="67375">chmod +x ./sum.sh</code>，给<code data-backticks="1" data-nodeid="67377">sum.sh</code>增加执行权限。然后我们可以像下图这样执行，获得结果：</p>
<p data-nodeid="67373" class=""><img src="https://s0.lgstatic.com/i/image/M00/5C/7F/CgqCHl-BkOKAfpNwAAOFk0EhDjU183.png" alt="Drawing 8.png" data-nodeid="67381"></p>


<p data-nodeid="54543">如上图，<code data-backticks="1" data-nodeid="54761">IP</code>地址已经按天进行统计好了。</p>
<h3 data-nodeid="54544">总结</h3>
<p data-nodeid="54545">今天我们结合一个简单的实战场景——Web 日志分析与统计练习了之前学过的指令，提高熟练程度。此外，我们还一起学习了新知识——功能强大的<code data-backticks="1" data-nodeid="54765">awk</code>文本处理语言。在实战中，我们对一个<code data-backticks="1" data-nodeid="54767">nginx</code>的<code data-backticks="1" data-nodeid="54769">access_log</code>进行了简单的数据分析，直观地获得了这个网站的访问情况。</p>
<p data-nodeid="54546">我们在日常的工作中会遇到各种各样的日志，除了 nginx 的日志，还有应用日志、前端日志、监控日志等等。你都可以利用今天学习的方法，去做数据分析，然后从中得出结论。</p>
<h3 data-nodeid="54547">思考题</h3>
<p data-nodeid="54548">接下来我给你出 2 个场景思考题，帮助你继续练习使用 Linux 指令。</p>
<ol data-nodeid="54549">
<li data-nodeid="54550">
<p data-nodeid="54551">根据今天的 access_log 分析出有哪些终端访问了这个网站，并给出分组统计结果。</p>
</li>
<li data-nodeid="54552">
<p data-nodeid="54553">根据今天的 access_log 分析出访问量 Top 前三的网页。</p>
</li>
</ol>
<p data-nodeid="67980">你可以把你的答案、思路或者课后总结写在留言区，这样可以帮助你产生更多的思考，这也是构建知识体系的一部分。经过长期的积累，相信你会得到意想不到的收获。如果你觉得今天的内容对你有所启发，欢迎分享给身边的朋友。期待看到你的思考！</p>

---

### 精选评论

##### *程：
> 收获：终于大概知道市面的埋点的PV和UV，怎么得来的了。手动狗头

##### **锋：
> 1. awk -F\" '{print $6}' access.log | sort | uniq -c | sort -fr2. awk '{print $7}' access.log | sort | uniq -c | sort -fr | head -3

##### **鹏：
> 第六步应该写错了，应该是:END{for (date in uv) print date, uv[date]}

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个代码在ubuntu中可以执行的。

##### *浩：
> 最近在做部署相关的东西，老师讲的这些很有用。

##### **飞：
> 第二题awk '{uv[$7]++;next}END{for (ip in uv)print ip,uv[ip]}' access. log | sort -rn -k 2|head -n 3

##### **康：
> 2.awk '{print $7}' access.log | sort | uniq -c | sort -fr | head -3

##### **康：
> 1. awk -F\" '{ print $6 }' access.log |awk '{print $1}'|awk -F/ '{print $1}'|sort|uniq -c|sort -fr|head -3

##### **俊：
> 收货满满，谢谢！

##### **用户9719：
> — — — —awk '{print $1}' ./access.log | sort | uniq -cawk '{print

