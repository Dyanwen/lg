<p data-nodeid="16426">我们在项目中经常会打印各种 Log 信息，用来查看程序运行中的详细情况。因此打印日志俨然已经成了程序开发工作中的一部分，而设计一个合理的 LogUtils 也成了一个好的程序员的必选条件之一。</p>



<h3 data-nodeid="16015">设置 Debug 开关</h3>
<p data-nodeid="16956">有时候为了调试方便，我们甚至会将用户的账号、密码、余额等信息也打印到控制台，但是如果这部分 Log 信息也出现在线上版本中，那用户的私密信息，或者程序相关核心实现都会被暴露；除此之外，打印日志的代码并不属于业务需求的必要代码，复杂的 Log 信息还会造成一定的性能损耗，所以这部分代码都不应该出现在线上版本的 App 中。因此我们需要设置一个开关来控制是否打印 Log 日志，只有在 Debug 版本才会打开此开关，如图所示：</p>
<p data-nodeid="16957" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/32/Ciqc1F79yuOAKRc0AACn5J73xWc380.png" alt="Drawing 0.png" data-nodeid="16961"></p>


<p data-nodeid="17490">通常情况下我们会使用 BuildConfig.DEBUG 来作为是否要打印日志的开关。但是使用这个变量具有一定的局限性。比如现场突然发现一个异常现象，而我们需要现场抓取异常的日志信息加以分析。因为是 release 版本，所有不会有任何 log 信息被打印。因此这个开关的设置最好具有一定的灵活性，比如可以再加一层 System Property 的设置，如图所示：</p>
<p data-nodeid="17491" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/32/Ciqc1F79yuuASf4RAAEC0G7d1i8786.png" alt="Drawing 1.png" data-nodeid="17495"></p>


<p data-nodeid="18024">上述代码打印结果如下所示：</p>
<p data-nodeid="18025" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/32/Ciqc1F79yvGAWzgGAAAydEsKlLQ971.png" alt="Drawing 2.png" data-nodeid="18029"></p>


<p data-nodeid="16022">使用 System Property 的好处是一旦设置之后，即使重启 App，System Property 中的变量依旧是设置之后的值，与 Android 中的 SharedPreference 非常相似。开发者只要定义好通过何种方式将这种属性打开即可，建议仿照 Android 系统设置中的“开发者选项”来实现，当用户快速连续点击某 item 时，才将此属性打开。</p>
<p data-nodeid="18558">另外，我们还可以通过 ProGuard 在打包阶段清除某些 Log 日志、打印代码，具体规则如下：</p>
<p data-nodeid="18559" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/3D/CgqCHl79yviAcvLDAABHPRbfCl0210.png" alt="Drawing 3.png" data-nodeid="18563"></p>


<h3 data-nodeid="16025">设置 log 日志本地保存</h3>
<p data-nodeid="19092">有时候我们需要将部分 log 日志以文件的形式保存在手机磁盘中，因此我们还需要设置开关，控制日志是打印在控制台还是保存到文件中。如下所示：</p>
<p data-nodeid="19093" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/32/Ciqc1F79ywGAUMaFAAHDevPE_DI210.png" alt="Drawing 4.png" data-nodeid="19097"></p>


<p data-nodeid="19626">因为涉及文件的写操作，所以最好是在子线程中完成日志的保存。因此在 LogUtils 中可以使用线程池控制子线程完成日志保存，如下所示：</p>
<p data-nodeid="19627" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/3D/CgqCHl79ywiAETP_AAI_AuTvNC0527.png" alt="Drawing 5.png" data-nodeid="19631"></p>


<h3 data-nodeid="16030">Config 文件统一配置</h3>
<p data-nodeid="20160">如果 LogUtils 中的开关较多，再加上还有其他配置项，比如日志保存为文件的路径等。这种情况可以使用一个全局的 Config 来配置 LogUtils 中所有的配置项，如下所示：</p>
<p data-nodeid="20161" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/32/Ciqc1F79yxCAZsaGAALTVgPeDRg031.png" alt="Drawing 6.png" data-nodeid="20165"></p>


<h4 data-nodeid="16033">特殊格式转换</h4>
<p data-nodeid="20694">我们经常会处理一些特殊格式的数据，比如 JSON、XML。为了打印这部分数据，还需要在 LogUtils 类中做一些格式转换的操作：</p>
<p data-nodeid="20695" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/3D/CgqCHl79yxmAaF8EAAKcSdHKQ3Y185.png" alt="Drawing 7.png" data-nodeid="20699"></p>

<h4 data-nodeid="20434">借助于三方库打印 log</h4>



<p data-nodeid="16038">如果感觉自己封装一个 LogUtils 类比较麻烦，或者没有好的实现思路，那就不妨尝试使用行内内已经成熟的 log 日志库。</p>
<p data-nodeid="16039"><strong data-nodeid="16116">XLog</strong></p>
<p data-nodeid="16040">XLog 是比较常用的打印日志开源库，GitHub 地址参考 <a href="https://github.com/elvishew/XLog/blob/master/README_ZH.md" data-nodeid="16120">XLog github</a>。XLog 基本囊括了我们上文介绍的所有功能：</p>
<ul data-nodeid="16041">
<li data-nodeid="16042">
<p data-nodeid="16043">全局配置或基于单条日志的配置；</p>
</li>
<li data-nodeid="16044">
<p data-nodeid="16045">支持打印任意对象以及可自定义的对象格式化器；</p>
</li>
<li data-nodeid="16046">
<p data-nodeid="16047">支持打印数组；</p>
</li>
<li data-nodeid="16048">
<p data-nodeid="16049">支持打印无限长的日志（没有 4K 字符的限制）；</p>
</li>
<li data-nodeid="16050">
<p data-nodeid="16051">XML 和 JSON 格式化输出；</p>
</li>
<li data-nodeid="16052">
<p data-nodeid="16053">线程信息（线程名等，可自定义）；</p>
</li>
<li data-nodeid="16054">
<p data-nodeid="16055">调用栈信息（可配置的调用栈深度，调用栈信息包括类名、方法名、文件名和行号）；</p>
</li>
<li data-nodeid="16056">
<p data-nodeid="16057">支持日志拦截器；</p>
</li>
<li data-nodeid="16058">
<p data-nodeid="16059">保存日志文件（文件名和自动备份策略可灵活配置）。</p>
</li>
</ul>
<p data-nodeid="21224">XLog 使用比较简单，先调用 init 方法进行初始化，最好是在 Application 中。</p>
<p data-nodeid="21225" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/32/Ciqc1F79yyOAXOcKAAFJcacSgv8868.png" alt="Drawing 8.png" data-nodeid="21229"></p>


<p data-nodeid="21754">然后就可以直接调用 Xlog 的静态方法打印相应日志即可：</p>
<p data-nodeid="21755" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/3D/CgqCHl79yyqATI2zAACuVSGDEV8600.png" alt="Drawing 9.png" data-nodeid="21759"></p>


<p data-nodeid="22284">也可以在打印日志时，添加局部的配置信息：</p>
<p data-nodeid="22285" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/3D/CgqCHl79yzmAZ3OIAABv32_jBIU173.png" alt="Drawing 10.png" data-nodeid="22289"></p>


<p data-nodeid="22814">打印结果类似下图所示：</p>
<p data-nodeid="22815" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/32/Ciqc1F79yz-AQZIZAADeZb8f9-Y734.png" alt="Drawing 11.png" data-nodeid="22819"></p>


<p data-nodeid="16068">可以看出，除了打印日志的类和方法，XLog 还能打印线程信息以及调用栈信息。</p>
<h3 data-nodeid="16069">总结</h3>
<p data-nodeid="23086">这节课主要介绍了项目开发中对 LogUtils 类的配置，因为项目中会在很多地方打印各种日志信息，所以为了方便统一管理，我们应该将所有日志打印的工作都集中到一个 Utils 类中。LogUtils 应该对外部提供相应的开关，用来设置是否需要打印日志，以及打印日志的通道，如果是将日志保存在文件中等耗时操作，还应该考虑在子线程中完成。</p>

---

### 精选评论

##### MilkBread：
> 老师，跨进程通信IPC比如binder机制可以讲一下吗？面试经常问到。市面上的资料很多看不懂，我觉得您讲得更易懂

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 您的问题已收到，小编会反馈给讲师喔，也建议您加入咱们的学习群和大家一起讨论。

