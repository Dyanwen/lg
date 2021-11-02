<p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(73, 73, 73);">你好，我是你的数据结构课老师蔡元楠，欢迎进入第 04 课时的内容“链表在 Apache Kafka 中的应用”。</span><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">经过了前三讲的学习之后，我相信你已经对数组和链表有了比较好的了解了。那在这一讲中，我想和你分享一下，数组和链表结合起来的数据结构是如何被大量应用在操作系统、计算机网络，甚至是在 Apache 开源项目中的。</span></p>
<h3><p style="line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">如何重新设计定时器算法</span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">说到<strong>定时器（Timer）</strong>，你应该不会特别陌生。像我们写程序时使用到的 Java Timer 类，或者是在 Linux 中制定定时任务时所使用的 cron 命令，亦或是在 BSD TCP 网络协议中检测网络数据包是否需要重新发送的算法里，其实都使用了定时器这个概念。那在课程的开头，我想先问问你，如果让你来重新设计定时器算法的话，会如何设计呢？</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本质上，定时器的实现是依靠着计算机里的时钟来完成的。举个例子，假设时钟是每秒跳一次，那我们可以根据时钟的精度构建出 10 秒或者 1 分钟的定时器，但是如果想要构建 0.5 秒的定时器是无法做到的，因为计算机时钟最快也只能每一秒跳一次，所以即便当我们设置了 0.5 秒的定时器之后，本质上这个定时器也是只有 1 秒。当然了，在现实中，计算机里时钟的精度都是毫微秒（Nanosecond）级别的，也就是十亿分之一秒。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">那回到设计定时器这个算法中，一般我们可以把定时器的概念抽象成 4 个部分，它们分别是：</span></p>
<ol>
 <li><p style="line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>初始化定时器</strong>，规定定时器经过了多少单位时间之后超时，并且在超时之后执行特定的程序；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>删除定时器</strong>，终止一个特定的定时器；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>定时器超时进程</strong>，定时器在超时之后所执行的特定程序；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>定时器检测进程</strong>，假设定时器里的时间最小颗粒度为 T 时间，则每经过 T 时间之后都会执行这个进程来查看是否定时器超时，并将其移除。</span></p></li>
</ol>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">你可能会问，我们现在只学习了数组和链表这两种数据结构，难道就可以设计一个被如此广泛应用的定时器算法了吗？完全没问题的，那我们就由浅入深，一起来看看各种实现方法优缺点吧。下面的所有算法我们都假设定时器超时时间的最小颗粒度为 T。</span></p>
<h3><p style="line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>维护无序定时器列表</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">最简单粗暴的方法，当然就是直接用数组或者链表来维护所有的定时器了。从前面的学习中我们可以知道，在数组中插入一个新的元素所需要的时间复杂度是 O(N)，而在链表的结尾插入一个新的节点所需要的时间复杂度是 O(1)，所以在这里可以选择用链表来维护定时器列表。假设我们要维护的定时器列表如下图所示：</span></p>
<p style="line-height: 2;line-height: 115%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/6F/CgpOIF4EZ5GAGIysAAAwMX46TJ4905.png" style="width: 478px; height: 66px;"></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">它表示现在系统维护了 3 个定时器，分别会在 3T、T 和 2T 时间之后超时。如果现在用户又插入了一个新定时器，将会在 T 时间后超时，我们会将新的定时器数据结构插入到链表结尾，如下图所示：</span></p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;"><img src="https://s0.lgstatic.com/i/image3/M01/5B/6F/Cgq2xl4EZ5GAOmjhAAAojT2d1yQ153.png" style="width: 675px; height: 96px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; text-align: justify;">每次经过 T 时间之后，定时器检测进程都会从头到尾扫描一遍这个链表，每扫描到一个节点的时候都会将里面的时间减去 T，然后判断这个节点的值是否等于 0 了，如果等于 0 了，则表示这个定时器超时，执行定时器超时进程并删除定时器，如果不等于，则继续扫描下一个节点。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这种方法的好处是定时器的插入和删除操作都只需要 O(1) 的时间。但是每次执行定时器检测进程的时间复杂度为 O(N)。如果定时器的数量还很小时还好，如果当定时器有成百上千个的时候，定时器检测进程就会成为一个瓶颈了。</span></p>
<h3><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>维护有序定时器列表</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这种方法是上述方法的改良版本。我们可以还是继续维护一个定时器列表，与第一种方法不一样的是，每次插入一个新的定时器时，并不是将它插入到链表的结尾，而是从头遍历一遍链表，将定时器的超时时间按从小到大的顺序插入到定时器列表中。还有一点不同的是，每次插入新定时器时，并不是保存超时时间，而是根据当前系统时间和超时时间算出一个绝对时间出来。例如，当前的系统时间为 NowTime，超时时间为 2T，那这个绝对时间就为 NowTime + 2T。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">假设原来的有序定时器列表如下图所示：</span></p>
<p style="line-height: 2;line-height: 115%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/6F/CgpOIF4EZ5GANmf7AAE9ok2RoEk947.png" style="width: 500px; height: 251px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">当我们要插入一个新的定时器，超时的绝对时间算出为 25 Dec 2019 9:23:34，这时候我们会按照超时时间从小到大的顺序，将定时器插入到定时器列表的开头，如下图所示：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/6F/Cgq2xl4EZ5GAezpLAAG4uRLeH28330.png" style="width: 500px; height: 325px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">维护一个有序的定时器列表的好处是，每次执行定时器检测进程的时间复杂度为 O(1)，因为每次定时器检测进程只需要判断当前系统时间是否是在链表第一个节点时间之后了，如果是则执行定时器超时进程并删除定时器，如果不是则结束定时器检测进程。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这种方法的好处是执行定时器检测进程和删除定时器的时间复杂度为 O(1)，但因为要按照时间从小到大排列定时器，每次插入的时候都需要遍历一次定时器列表，所以插入定时器的时间复杂度为 O(N)。</span></p>
<h3><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>维护定时器“时间轮”</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>“时间轮”（Timing-wheel ）在概念上是一个用数组并且数组元素为链表的数据结构来维护的定时器列表，常常伴随着溢出列表（Overflow List）来维护那些无法在数组范围内表达的定时器。</strong>“时间轮”有非常多的变种，今天我来解释一下最基本的“时间轮”实现方式。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">首先基本的“时间轮”会将定时器的超时时间划分到不同的<strong>周期</strong><strong>（</strong><strong>Cycle</strong><strong>）</strong>中去，数组的大小决定了一个周期的大小。例如，一个“时间轮”数组的大小为 8，那这个“时间轮”周期的大小就为 8T。同时，我们维护一个最基本的“时间轮”还需要维护以下几个变量：</span></p>
<ul>
 <li><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">“时间轮”的周期数，用 S 来表示；</span></p></li>
 <li><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">“时间轮”的周期大小，用 N 来表示；</span></p></li>
 <li><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">“时间轮”数组现在所指向的索引，用 i 来表示。</span></p></li>
</ul>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">现在的时间我们可以用 S<span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(34, 34, 34);">×N + i 来表示</span>，每次我们执行完一次定时器检测进程之后，都会将 i 加 1。当 i 等于 N 的时候，我们将 S 加 1，并且将 i 归零。因为“时间轮”里面的数组索引会一直在 0 到 N－1 中循环，所以我们可以将数组想象成是一个环，例如一个“时间轮”的周期大小为 8 的数组，可以想象成如下图所示的环：</span></p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;"><img src="https://s0.lgstatic.com/i/image3/M01/5B/6F/CgpOIF4EZ5GAXlGMAABboVgG49c716.png" style="width: 300px; height: 241px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">那么我们假设现在的时间是 S<span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(34, 34, 34);">×N + 2，表示这个</span>“时间轮”的当前周期为 S，数组索引为 2，同时假设<span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(34, 34, 34);">这个</span>“时间轮”已经维护了一部分定时器链表，如下图所示：</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><br></p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;"><img src="https://s0.lgstatic.com/i/image3/M01/5B/6F/Cgq2xl4EZ5GAfv-zAAEx8tfoD5Q555.png" style="width: 306px; height: 120px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; text-align: justify;">如果我们想新插入一个超时时间为 T 的新定时器进这个时间轮，因为 T 小于这个“时间轮”周期的大小 8T，所以表示这个定时器可以被插入到当前的“时间轮”中，插入的位置为当前索引为 1 + 2 % 8 = 3 ，插入新定时器后的“时间轮”如下图所示：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; text-align: justify;"><br></span></p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;"><img src="https://s0.lgstatic.com/i/image3/M01/5B/6F/CgpOIF4EZ5GAE6MFAAFph4owXi0789.png" style="width: 300px; height: 118px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如果我们现在又想新插入一个超时时间为 9T 的新定时器进这个“时间轮”，因为 9T 大于或等于这个“时间轮”周期的大小 8T，所以表示这个定时器暂时无法被插入到当前的周期中，我们必须将这个新的定时器放进溢出列表里。溢出列表存放着新定时器还需要等待多少周期才能进入到当前“时间轮”中，我们按照下面公式来计算还需等待的周期和插入的位置：</span></p>
<pre>还需等待的周期：9T&nbsp;/&nbsp;8T&nbsp;=&nbsp;1
新定时器插入时的索引位置：(9T&nbsp;+&nbsp;2T)&nbsp;%&nbsp;8T&nbsp;=&nbsp;3</pre>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们算出了等待周期和新插入数组的索引位置之后，就可以更新溢出列表，如下图所示：</span></p>
<p style="line-height: 2;line-height: 115%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/6F/Cgq2xl4EZ5KAbE3FAAGRd4XWnFs829.png" style="width: 300px; height: 140px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在“时间轮”的算法中，定时器检测进程只需要判断“时间轮”数组现在所指向的索引里的链表为不为空，如果为空则不执行任何操作，如果不为空则对于这个数组元素链表里的所有定时器执行定时器超时进程。而每当“时间轮”的周期数加 1 的时候，系统都会遍历一遍溢出列表里的定时器是否满足当前周期数，如果满足的话，则将这个位置的溢出列表全部移到“时间轮”相对应的索引位置中。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在这种基本“时间轮”的算法里，定时器检测进程的时间复杂度为 O(1)，而插入新定时器的时间复杂度取决于超时时间，因为插入的新定时器有可能会被放入溢出列表中从而需要遍历一遍溢出列表以便将新定时器放入到相对应周期的位置。</span></p>
<h3><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>“时间轮”变种</strong><strong>算法</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">基本的“时间轮”插入操作因为维护了一个溢出列表导致定时器的插入操作无法做到 O(1) 的时间复杂度，所以为了 O(1) 时间复杂度的插入操作，一种变种的“时间轮”算法就被提出了。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在这个变种的“时间轮”算法里，我们加了一个 MaxInterval 的限制，这个 MaxInterval 其实也就是我们定义出的“时间轮”数组的大小。假设“时间轮”数组的大小为 N，对于任何需要新加入的定时器，如果超时时间小于 N 的话，则被允许加入到“时间轮”中，否则将不被允许加入。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这种“时间轮”变种算法，执行定时器检测进程还有插入和删除定时器的操作时间复杂度都只有 O(1)。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></span></p>
<h3><p><strong>分层“时间轮”</strong></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">上面所描述到的“时间轮”变种算法，当我们要表达的 MaxInterval 很大而且超时时间颗粒度比较小的时候，会占用比较大的空间。例如，如果时间颗粒度是 1 秒，而 MaxInterval 是 1 天的话，就表示我们需要维护一个大小为 24 × 60 × 60 = 86400 的数组。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;"><strong>那有没有方法可以将空间利用率提高，而同时保持着执行定时器检测进程还有插入和删除定时器的操作时间复杂度都只有</strong><strong> </strong><strong>O(1)</strong><strong> </strong><strong>呢？答案是使用分层“时间轮”</strong><strong>（</strong><strong>Hierarchical Timing Wheel</strong><strong>）</strong>。下面还是以时间颗粒度是 1 秒，而 MaxInterval 是 1 天的例子来说明分层“时间轮”算法。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">我们可以使用三个“时间轮”来表示不同颗粒度的时间，分别是小时“时间轮”、分钟“时间轮”和秒“时间轮”，可以称小时“时间轮”为分钟“时间轮”的上一层“时间轮”，秒“时间轮”为分钟“时间轮”的下一层“时间轮”。分层“时间轮”会维护一个“现在时间”，每层“时间轮”都需要各自维护一个当前索引来表示“现在时间”。例如，分层“时间轮”的“现在时间”是22h:20min:30s，它的结构图如下图所示：</span></p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/7A/CgpOIF4Efl2Ad4lgAAHYUt39Xk4150.png" style="width: 500px; height: 263px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">当每次有新的定时器需要插入进分层“时间轮”的时候，将根据分层“时间轮”的“现在时间”算出一个超时的绝对时间。例如，分层“时间轮”的“现在时间”是 22h:20min:30s，而当我们要插入的新定时器超时时间为 50 分钟 10 秒时，这个超时的绝对时间则为 23h:10min:40s。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">我们需要先判断最高层的时间是否一致，如果不一致的话则算出时间差，然后插入定时器到对应层的“时间轮”中，如果一致，则到下一层中的时间中计算，如此类推。在上面的例子中，最高层的时间小时相差了 23－22 = 1 小时，所以需要将定时器插入到小时“时间轮”中的 (1 + 21) % 24 = 22这个索引中，定时器列表里还需要保存下层“时间轮”所剩余的时间 10min:40s，如下图所示：</span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/7B/Cgq2xl4Efl2AFC8yAAH_xBDJGqM733.png" style="width: 500px; height: 258px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">每经过一秒钟，秒“时间轮”的索引都会加 1，并且执行定时器检测进程。定时器检测进程需要判断当前元素里的定时器列表是否为空，如果为空则不执行任何操作，如果不为空则对于这个数组元素列表里的所有定时器执行定时器超时进程。需要注意的是，定时器检测进程只会针对最下层的“时间轮”执行。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">如果秒“时间轮”的索引到达 60 之后会将其归零，并将上一层的“时间轮”索引加 1，同时判断上一层的“时间轮”索引里的列表是否为空，如果不为空，则按照之前描述的算法将定时器加入到下一层“时间轮”中去，如此类推。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">在经过一段时间之后，上面的分层“时间轮”会到达以下的一个状态：</span></p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/7A/CgpOIF4Efl2AM9mbAAH_RIwfKm8774.png" style="width: 500px; height: 251px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">这时候上层“时间轮”索引里的列表不为空，将这个定时器加入的索引为 10 的分钟“时间轮”中，并且保存下层“时间轮”所剩余的时间 40s，如下图所示：</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: center;"><span style="font-size: 16px;"> &nbsp; &nbsp; &nbsp; &nbsp;</span><img src="https://s0.lgstatic.com/i/image3/M01/5B/7B/Cgq2xl4Efl2AVj80AAHqhEPdsAo113.png" style="width: 600px; height: 322px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">如此类推，在经过 10 分钟之后，分层“时间轮”会到达以下的一个状态：</span></p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/7A/CgpOIF4Efl2ATWRDAAH0naPN-HA794.png" style="width: 500px; height: 253px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">同样的，我们将这个定时器插入到秒“时间轮”中，如下图所示：</span></p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/7B/Cgq2xl4Efl2AIm04AAHF7KzHJcs847.png" style="width: 500px; height: 243px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">这个时候，再经过 40 秒，秒“时间轮”的索引将会指向一个元素，里面有着非空的定时器列表，然后执行定时器超时进程并将定时器列表里所有的定时器删除。</span></p>
<p style="line-height: 115%; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;">&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/5B/7A/CgpOIF4Efl6AfQaHAAHWoZdqcY4094.png" style="width: 500px; height: 250px;"> &nbsp; &nbsp; &nbsp;</p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;">&nbsp;</p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px;">我们可以看到，采用了分层“时间轮”算法之后，我们只需要维护一个大小为 24 + 60 + 60 = 144 的数组，而同时保持着执行定时器检测进程还有插入和删除定时器的操作时间复杂度都只有 O(1)。</span></p>
<p><br></p>
<h3><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>Apache Kafka</strong><strong> </strong><strong>的</strong><strong> </strong><strong>Purgatory</strong><strong> </strong><strong>组件</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Apache Kafka 是一个开源的消息系统项目，主要用于提供一个实时处理消息事件的服务。与计算机网络里面的 TCP 协议需要用到大量定时器来判断是否需要重新发送丢失的网络包一样，在 Kafka 里面，因为它所提供的服务需要判断所发送出去的消息事件是否被订阅消息的用户接收到，Kafka 也需要用到大量的定时器来判断发出的消息是否超时然后重发消息。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">而这个任务就落在了 Purgatory 组件上。在旧版本的 Purgatory 组件里，维护定时器的任务采用的是 Java 的 DelayQueue 类来实现的。DelayQueue 本质上是一个堆（Heap）数据结构，这个概念将会在第 09 讲中详细介绍。现在我们可以把这种实现方式看作是维护有序定时器列表的一种变种。这种操作的一个缺点是当有大量频繁的插入操作时，系统的性能将会降低。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">因为 Kafka 中所有的最大消息超时时间都已经被写在了配置文件里，也就是说我们可以提前知道一个定时器的 MaxInterval，所以新版本的 Purgatory 组件则采用的了我们上面所提到的变种“时间轮”算法，将插入定时器的操作性能大大提升。根据 Kafka 所提供的检测结果，采用 DelayQueue 时所能处理的最大吞吐率为 25000 RPS，采用了变种“时间轮”算法之后，最大吞吐率则达到了 105000 RPS。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">OK，这节课就讲到这里啦，下一课时我将分享“哈希函数的本质及生成方式”，记得按时来听课哈。</span></p>

---

### 精选评论

##### **杰：
> 表示看懵了😲

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 很棒哦

##### **俊：
> 这个分层时间轮，灵感来源于生活呀，是古老的智慧。“当我们给自己设定一个时间去做某件事的时候。比如说，14:40:40我要睡觉。如果现在是13点，那还没到14点，我会把这个任务放到14点的位置，保留40:40。当时间变成14点的时候。我会想到40分的时候我需要睡觉了，这时我会把任务放到40分，保留40s。当时间变成14:40时，我还有40s就要睡觉了。我会把任务放到秒的数组中39的位置。终于时间变成14:40:40时，我会准时睡觉，并把这个定时任务标记为已完成。”

##### zjrc-enterc104：
> 有个问题：分层时间轮中，新加超时的绝对时间则为 23h:10min:40s超时定时器时，先加到22h的小时“定时轮”中，这个可以理解，因为23h的索引为22，但为什么在小时“时间轮”索引到22时将23h:10min:40s这个定时器加到分钟“时间轮”索引为10中，而不是索引为9中？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里采用的公式是(Current Index + 剩余时间) % 时间轮周期，也就是（0 + 10）% 60。您可以简化这个例子一下，如果现在剩余的时间只有1min:0s的话，把这个剩余时间加到索引为0的位置就相当于已经超时了。

##### **7678：
> 您好，“时间轮”变种算法中的，“执行定时器检测进程还有插入和删除定时器的操作时间复杂度都只有 O(1)”这句话错了吧？如果插入的是时间轮时间复杂度为O(1)，如果插入到溢出列表时间复杂度就不是O(1)了。所以这里是错的。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢这位同学的提问！这一讲中提到的“时间轮”变种算法其实是加了一些限制，也就是定时器的超时时长不会超过“时间轮”数组的大小，所以这里也就没有插入到溢出列表的情况了。一般来说这种变种算法适合一些定时器大小比较固定且都比较小的应用场景。

##### *宁：
> 在时间轮里面插入操作是O(1)这个可以理解，因为相同绝对超时时间的定时器放在同一个链表里面，直接插入到链表尾部就好了，但是删除操作怎么也会是O(1)呢？如果它在链表的中间，不是需要遍历链表找出定时器所在的位置么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个问题很好！在“维护无序定时器列表“这一内容的上下文中，删除一个定时器发生的条件是定时器检测进程发现某个定时器超时了，执行定时器超时进程并删除定时器，所以这里的删除操作时间复杂度就是O(1)。当然了，如果仅针对链表的删除操作，从查找某个特定定时器到删除这一步，时间复杂度会是O(N)。

##### *帅：
> 讲的确实很好，高屋建瓴，但是更多的是从原理来讲解，要是有真实的源代码分析就更好了

##### **亮：
> kafka有了maxinterval，是不是理解是，超时的消息会一直重发，直到超过了maxinterval。譬如超过10分钟就重发，然后maxinterval是20分钟，最多只会重试10分钟，然后超时也不管的意思吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢这位同学的提问！Kafka 里的 maxinterval 指的是超过了多少时间会进行重发，而定义重发次数是由其他 configurations 来定义 retry 次数的。

##### **红：
> ”如果超时时间小于 N 的话，则被允许加入到“时间轮”中，否则将不被允许加入“，这里那这个定时器会什么怎么保存或处理，然后在什么时候加入呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢这位同学的提问！变种定时器的处理方式其实和“维护定时器时间轮”这一章节中所讲述的一模一样，唯一的区别就是有了MaxInterval 的限制，也就是不用去维护一个溢出列表了。一般来说这种变种算法适合一些定时器大小比较固定且都比较小的应用场景。

##### *毅：
> 如何实现粒度小于秒级的定时器呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 课程里面的例子仅是为了方便理解所以使用了以1秒作单位。在现实中，能够实现怎么样的时间颗粒度定时器还取决于系统能够支持的最小时间颗粒度，像现代的计算机基本都至少能支持毫秒（Millisecond）级的定时器。

##### Alps：
> 时间轮作为链表的应用，讲的很清楚，也对链表有了更深一层的理解了

##### **涛：
> 真的讲的太好了！这些内容对我们移动端开发来说也是非常有用的。

