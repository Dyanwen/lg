<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">本课时我们主要学习线程池各个参数的含义，并重点掌握线程池中线程是在什么时机被创建和销毁的。</span></p> 
<h2 style="box-sizing: border-box; color: rgb(0, 0, 0); font-family: -apple-system,BlinkMacSystemFont,PingFang SC,Helvetica,Tahoma,Arial,&amp;quot;Hiragino Sans GB&amp;quot;,&amp;quot;Microsoft YaHei&amp;quot;,&amp;quot;微软雅黑&amp;quot;,sans-serif; font-size: 22px; font-style: normal; font-variant: normal; font-weight: 700; letter-spacing: normal; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; margin-top: 0px; orphans: 2; outline-color: invert; outline-style: none; outline-width: medium; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;"><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">线程池的参数</span></p></h2> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><img src="https://s0.lgstatic.com/i/image2/M01/AD/A3/CgoB5l3eH8mAAoJCAACEOKMHtpw036.png"></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">首先，我们来看下线程池中各个参数的含义，如表所示线程池主要有 6 个参数，其中第 3 个参数由 keepAliveTime + 时间单位组成。我们逐一看下它们各自的含义，</span><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box; background-color: rgb(255, 255, 255);">corePoolSize 是核心线程数，也就是常驻线程池的线程数量，与它对应的是&nbsp;</span></span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">，表示线程池最大线程数量，当我们的任务特别多而 corePoolSize 核心线程数无法满足需求的时</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">候，就会向线程池中增加线程，以便应对任务突增的情况。</span></p> 
<h2 style="box-sizing: border-box; color: rgb(0, 0, 0); font-family: -apple-system,BlinkMacSystemFont,PingFang SC,Helvetica,Tahoma,Arial,&amp;quot;Hiragino Sans GB&amp;quot;,&amp;quot;Microsoft YaHei&amp;quot;,&amp;quot;微软雅黑&amp;quot;,sans-serif; font-size: 22px; font-style: normal; font-variant: normal; font-weight: 700; letter-spacing: normal; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; margin-top: 0px; orphans: 2; outline-color: invert; outline-style: none; outline-width: medium; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;"><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">线程创建的时机</span></p></h2> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;"><img src="https://s0.lgstatic.com/i/image6/M00/58/0F/Cgp9HWE7GMmAZ_OzAADmBM70EkI437.png" style="text-align: left; color: rgb(51, 51, 51); max-width: 100%;">&nbsp; &nbsp; &nbsp;&nbsp;</span></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">接下来，我们来具体看下这两个参数所代表的含义，以及线程池中创建线程的时机。如上图所示，当提交任务后，线程池首先会检查当前线程数，如果此时线程数小于核心线程数，比如最开始线程数量为 0，则新建线程并执行任务，随着任务的不断增加，线程数会逐渐增加并达到核心线程数，此时如果仍有任务被不断提交，就会被放入 workQueue 任务队列中，等待核心线程执行完当前任务后重新从 workQueue 中提取正在等待被执行的任务。</span></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;">&nbsp;</span></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;"><span style="margin: 0px; padding: 0px; outline: invert; font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">此时，假设我们的任务特别的多，已经达到了 workQueue 的容量上限，这时线程池就会启动后备力量，也就是&nbsp;</span></span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;最大线程数，线程池会在 corePoolSize 核心线程数的基础上继续创建线程来执行任务，假设任务被不断提交，线程池会持续创建线程直到线程数达到&nbsp;</span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="font-style: normal; font-variant-caps: normal; color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;最大线程数，如果依然有任务被提交，这就超过了线程池的最大处理能力，这个时候线程池就会拒绝这些任务，我们可以看到实际上任务进来之后，线程池会逐一判断 corePoolSize、</span><span style="font-style: normal; font-variant-caps: normal; color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">workQueue</span><span style="font-style: normal; font-variant-caps: normal; color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">、</span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; font-style: normal; font-variant-caps: normal;">，如果依然不能满足需求，则会拒绝任务。</span></p> 
<h2 style="box-sizing: border-box; color: rgb(0, 0, 0); font-family: -apple-system,BlinkMacSystemFont,PingFang SC,Helvetica,Tahoma,Arial,&amp;quot;Hiragino Sans GB&amp;quot;,&amp;quot;Microsoft YaHei&amp;quot;,&amp;quot;微软雅黑&amp;quot;,sans-serif; font-size: 22px; font-style: normal; font-variant: normal; font-weight: 700; letter-spacing: normal; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; margin-top: 0px; orphans: 2; outline-color: invert; outline-style: none; outline-width: medium; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;"><p style="text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;"><span style="margin: 0px; padding: 0px; outline: invert; font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box; background-color: rgb(255, 255, 255);">corePoolSize 与 maximumPoolSize</span><span style="margin: 0px; padding: 0px; outline: invert; font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">&nbsp; &nbsp;</span></span></p></h2> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;"><span style="margin: 0px; padding: 0px; outline: invert; font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">通过上面的流程图，我们了解了 </span><span style="margin: 0px; padding: 0px; outline: invert; font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box; background-color: rgb(255, 255, 255);">corePoolSize 和&nbsp;</span></span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">&nbsp;的具体含义，corePoolSize 指的是核心线程数，线程池初始化时线程数默认为 0，当有新的任务提交后，会创建新线程执行任务，如果不做特殊设置，此后线程数通常不会再小于 corePoolSize ，因为它们是核心线程，即便未来可能没有可执行的任务也不会被销毁。随着任务量的增加，在任务队列满了之后，线程池会进一步创建新线程，最多可以达到&nbsp;</span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;来应对任务多的场景，如果未来线程有空闲，大于 corePoolSize 的线程会被合理回收。所以正常情况下，线程池中的线程数量会处在 corePoolSize 与&nbsp;</span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;的闭区间内。</span></p> 
<h2 style="box-sizing: border-box; color: rgb(0, 0, 0); font-family: -apple-system,BlinkMacSystemFont,PingFang SC,Helvetica,Tahoma,Arial,&amp;quot;Hiragino Sans GB&amp;quot;,&amp;quot;Microsoft YaHei&amp;quot;,&amp;quot;微软雅黑&amp;quot;,sans-serif; font-size: 22px; font-style: normal; font-variant: normal; font-weight: 700; letter-spacing: normal; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; margin-top: 0px; orphans: 2; outline-color: invert; outline-style: none; outline-width: medium; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;"><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box; background-color: rgb(255, 255, 255);">“长工”与“临时工”</span></p></h2> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;"><span style="margin: 0px; padding: 0px; outline: invert; font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">我们可以把 </span><span style="margin: 0px; padding: 0px; outline: invert; font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box; background-color: rgb(255, 255, 255);">corePoolSize 与&nbsp;</span></span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">&nbsp;比喻成长工与临时工，通常古代一个大户人家会有几个固定的长工，负责日常的工作，而大户人家起初肯定也是从零开始雇佣长工的。假如长工数量被老爷设定为 5 人，也就对应了 corePoolSize，不管这 5 个长工是忙碌还是空闲，都会一直在大户人家待着，可到了农忙或春节，长工的人手显然就不够用了，这时就需要雇佣更多的临时工，这些临时工就相当于在 corePoolSize 的基础上继续创建新线程，但临时工也是有上限的，也就对应了&nbsp;</span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">，随着农忙或春节结束，老爷考虑到人工成本便会解约掉这些临时工，家里工人数量便会从&nbsp;</span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;降到 corePoolSize，所以老爷家的工人数量会一致保持在 corePoolSize 和&nbsp;</span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;的区间。</span></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><img src="https://s0.lgstatic.com/i/image2/M01/AD/C4/CgotOV3eIA2AY8DaAC4VmOi19V8654.gif"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">&nbsp; &nbsp; &nbsp;</span></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;"><span style="margin: 0px; padding: 0px; outline: invert; font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">在这里我们用一个动画把整个线程池变化过程生动地描述出来，比如线程池的 </span><span style="margin: 0px; padding: 0px; outline: invert; font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box; background-color: rgb(255, 255, 255);">corePoolSize 为 5，</span></span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;为 10，任务队列容量为 100，随着任务被提交，我们的线程数量会从 0 慢慢增长到 5，然后就不再增长了，新的任务会被放入队列中，直到队列被塞满，然后在 corePoolSize 的基础上继续创建新线程来执行队列中的任务，线程会逐渐增加到&nbsp;</span><span style="text-align: left; color: rgb(51, 51, 51);">maximumPoolSize</span><span style="font-style: normal; font-variant-caps: normal; color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">， 然后线程数不再增加，如果此时仍有任务被不断提交，线程池就会拒绝任务。随着队列中任务被执行完，被创建的 10 个线程现在无事可做了，这时线程池会根据 </span><span style="font-style: normal; font-variant-caps: normal; color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">keepAliveTime 参数来销毁线程，已达到减少内存占用的目的。</span></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;">&nbsp;<span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;"> &nbsp; &nbsp; &nbsp;</span></span></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">通过对流程图的理解和动画演示，我们总结出线程池的几个特点。</span></p> 
<ul style="list-style-type: disc;"> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">线程池希望保持较少的线程数，并且只有在负载变得很大时才增加线程。</span></p></li> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">线程池只有在任务队列填满时才创建多于 corePoolSize 的线程，如果使用的是无界队列（例如 LinkedBlockingQueue），那么由于队列不会满，所以线程数不会超过 corePoolSize。</span></p></li> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">通过设置 corePoolSize 和&nbsp;</span><span style="text-align: left;">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;为相同的值，就可以创建固定大小的线程池。</span></p></li> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">通过设置&nbsp;</span><span style="text-align: left;">maximumPoolSize</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;为很高的值，例如 Integer.MAX_VALUE，就可以允许线程池创建任意多的线程。</span></p></li> 
</ul> 
<h2 style="box-sizing: border-box; color: rgb(0, 0, 0); font-family: -apple-system,BlinkMacSystemFont,PingFang SC,Helvetica,Tahoma,Arial,&amp;quot;Hiragino Sans GB&amp;quot;,&amp;quot;Microsoft YaHei&amp;quot;,&amp;quot;微软雅黑&amp;quot;,sans-serif; font-size: 22px; font-style: normal; font-variant: normal; font-weight: 700; letter-spacing: normal; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; margin-top: 0px; orphans: 2; outline-color: invert; outline-style: none; outline-width: medium; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;"><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">keepAliveTime+时间单位 &nbsp; &nbsp;&nbsp;</span></p></h2> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">第三个参数是 keepAliveTime + 时间单位，当线程池中线程数量多于核心线程数时，而此时又没有任务可做，线程池就会检测线程的 keepAliveTime，如果超过规定的时间，无事可做的线程就会被销毁，以便减少内存的占用和资源消耗。如果后期任务又多了起来，线程池也会根据规则重新创建线程，所以这是一个可伸缩的过程，比较灵活，我们也可以用 setKeepAliveTime 方法动态改变 keepAliveTime 的参数值。 </span></p> 
<h2 style="box-sizing: border-box; color: rgb(0, 0, 0); font-family: -apple-system,BlinkMacSystemFont,PingFang SC,Helvetica,Tahoma,Arial,&amp;quot;Hiragino Sans GB&amp;quot;,&amp;quot;Microsoft YaHei&amp;quot;,&amp;quot;微软雅黑&amp;quot;,sans-serif; font-size: 22px; font-style: normal; font-variant: normal; font-weight: 700; letter-spacing: normal; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; margin-top: 0px; orphans: 2; outline-color: invert; outline-style: none; outline-width: medium; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;"><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">ThreadFactory &nbsp;&nbsp;</span></p></h2> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">第四个参数是 ThreadFactory，ThreadFactory 实际上是一个线程工厂，它的作用是生产线程以便执行任务。我们可以选择使用默认的线程工厂，创建的线程都会在同一个线程组，并拥有一样的优先级，且都不是守护线程，我们也可以选择自己定制线程工厂，以方便给线程自定义命名，不同的线程池内的线程通常会根据具体业务来定制不同的线程名。</span></p> 
<h2 style="box-sizing: border-box; color: rgb(0, 0, 0); font-family: -apple-system,BlinkMacSystemFont,PingFang SC,Helvetica,Tahoma,Arial,&amp;quot;Hiragino Sans GB&amp;quot;,&amp;quot;Microsoft YaHei&amp;quot;,&amp;quot;微软雅黑&amp;quot;,sans-serif; font-size: 22px; font-style: normal; font-variant: normal; font-weight: 700; letter-spacing: normal; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; margin-top: 0px; orphans: 2; outline-color: invert; outline-style: none; outline-width: medium; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;"><p style="text-align: justify; line-height: 1.75em;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">workQueue 和 Handler &nbsp; &nbsp;&nbsp;</span></p></h2> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">最后两个参数是 workQueue 和 Handler，它们分别对应阻塞队列和任务拒绝策略，在后面的课时会对它们进行详细</span><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box; background-color: rgb(255, 255, 255);">展开</span><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">讲解。</span></span></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px;">&nbsp;</span></p> 
<p style="margin: 0pt 0px; padding: 0px; outline: invert; text-align: justify; color: rgb(73, 73, 73); text-transform: none; line-height: 1.75em; text-indent: 0px; letter-spacing: normal; font-size: 11pt; font-style: normal; font-variant: normal; font-weight: 400; text-decoration: none; word-spacing: 0px; white-space: normal; box-sizing: border-box; orphans: 2; -webkit-text-stroke-width: 0px;"><span style="margin: 0px; padding: 0px; outline: invert; color: rgb(63, 63, 63); font-family: 微软雅黑,Microsoft YaHei; font-size: 16px; box-sizing: border-box;">在本课时，介绍了线程池的各个参数的含义，以及如果有任务提交，线程池是如何应对的，新线程是在什么时机下被创建和销毁等内容，你有没有觉得线程池的设计很巧妙呢？</span><br></p> 
<p><br></p>

---

### 精选评论

##### **生：
> 讲得太牛逼了😀

##### *勇：
> keepAliveTime+时间单位会不会销毁核心线程了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 默认不会，不过可以配置成连核心线程也销毁。

##### *达：
> 睡觉前看一看，有一个问题，假如核心线程数10个，目前已创建好了5个线程，其中有俩个线程是空闲的，然后这个时候又进来一个任务，是会创建新的核心线程还是使用空闲的核心线程？希望老师回复下，谢谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 创建新的核心线程。

##### Aze：
> 如果一个线程执行任务的时间足够久，超过了KeepAliveTime，那还会被销毁吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 执行任务的时间不算，只计算空闲时的。

##### **鹏：
> 初始化线程池中线程数是0，有任务时创建线程来执行，是一下子创建n(核心线程数)个线程还是从0到n？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 默认是根据任务的到来，慢慢创建线程数直到核心线程数。

##### **才：
> 假设，核心线程已满，工作队列已满，新进来任务，这时候会创建非核心线程，线程总数量刚好到达maximumPoolSize，这时候，核心线程和非核心线程的执行顺序，有没有优先级，如果这个时候，非核心线程还有空闲，但核心线程占用，工作队列里面的任务会通过非核心线程执行，还是等待核心线程有空闲线程的时候，进入到核心线程。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 会通过非核心线程执行

##### **凯：
> 已经达到核心线程数，队列已满，这样会创建新线程执行任务，请问最新的这个任务是马上用新线程执行吗？还是取队列头的一个任务用新建立的线程执行，把这个最新的任务放到队列尾部？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 需要排队

##### *飞：
> 假如核心线程数10个，目前已创建好了5个线程，其中有俩个线程是空闲的，然后这个时候又进来一个任务，是会创建新的核心线程还是使用空闲的核心线程？为什么不使用空闲的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 创建新的，因为线程池希望快速把线程数扩展到核心线程数。

##### **祥：
> 讲的很棒

##### **亮：
> keepAliveTime 参数来销毁线程，目前是用来销毁“临时工”的，也能用来销毁“长工”吗 如何配置

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Java 核心线程池的回收由allowCoreThreadTimeOut参数控制，默认为false，若开启为true，则此时线程池中不论核心线程还是非核心线程，只要其空闲时间达到keepAliveTime都会被回收。但如果这样就违背了线程池的初衷（减少线程创建和开销），所以默认该参数为false。

##### *启：
> 2021-1-4晚学习，总算静下心来看这个课程了，对于线程池的理解，更加深一层，也总算是明白了各个参数的意义，以及线程池的执行流程

##### *桃：
> 如果使用无界队列，如果任务堆积得越来越多，而且可能响应还会越来越慢。最终的结果应该是 OOM 吧！有一个问题，为什么是队列满了才创建多于核心线程数的线程来处理任务，而不是某个阈值呢？比如当队列数量达到百分之八十的时候，就创建多于核心数的线程来处理任务，这样会不会能更好的防止队列阻塞，提高响应速度呢？在线程池中一个线程是核心线程还是非核心线程有严格的区分吗？希望以往为题能得到老师的解答，非常感谢。

##### *裕：
> 很清晰，核心线程数、最大线程数、工作队列、keepAlive+Unit 、线程工厂、拒绝策略

##### **顺：
> 之前一直以为核心线程数满了-队列，还好今天看到老师的文章了！！！😘

##### **云：
> 看 老师解答别人的问题时，您是怎么知道这么底层的呢，是要把源码都看 一遍吗，还是看书

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 两者结合

##### **晨：
> 线程池的参数讲得太好了~

##### **0028：
> 精干，易懂。

##### **栓：
> 哈哈哈，看完这一篇终于懂了线程池的原理了，非常感谢！！

##### **啦啦：
> 老师讲的很透彻

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 谢谢支持～

##### **利：
> 老师线程池的拒绝策略是 线程数达到最大线程数并且任务队列已经满了 这时候再增加任务就会被拒绝是吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 有两种拒绝时机，你说的是第一种，第二种是线程池已经被关闭。

##### **滔：
> corePoolSize：核心线程数（“长工”），常驻线程的数量。随着任务增多，线程池从0开始增加。maxPoolSize：最大线程数，创建线程的最大容量。是核心线程数与非核心线程数之和。keepAliveTime+时间单位：空闲线程存活时间。当非核心线程（“临时工”）空闲时，过了存活时间该线程就会被回收 。ThreadFactory：创建线程的工厂。workQueue：存放任务的队列。任务队列满了，会创建非核心线程，直至达到最大线程数。Handler：任务拒绝策略。当线程数达到最大，并且队列被塞满时，会拒绝任务。

##### **彪：
> 看完点赞，彪

##### *桃：
> 线程池参数:核心线程数，最大线程数，存活时间，线程工厂，任务队列，拒绝策略

##### *松：
> 打卡

##### **兵：
> 如果核心线程数等于最大线程数，当没有任务切超过设定的时间，核心线程数也会被回收吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不会的

##### Aze：
> 假设workQueue已满，此时有新任务到来，扩容到maxSize的过程中，新到的这个任务是进入queue等待还是直接可以执行

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 需要排队

##### *亮：
> 核心线程数是启动线程池的时候创建等待任务,还是等第一个任务到来,一下子创建核心线程数数量的线程,还是根据任务数慢慢创建线程数直到核心线程数<div><br></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 默认是根据任务慢慢创建线程数直到核心线程数，但是可以通过参数调节具体的策略。

##### **4347：
> 老师对于使用了无界队列 那么最大线程数是不是就不会用到了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 通常情况下，如果使用了无界队列，那么队列不会满，所以用不到最大线程数。

##### **帅：
> 线程池是怎么自动回收超时的线程的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在runWorker方法中，会调用processWorkerExit方法，里面会移除worker。

