<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="background-color: rgb(255, 255, 255); color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在本课时我们主要讲解 Shell 命令合集，以及对控制台的使用技巧。在正式学习这个课时之前你需要掌握如下三部分的知识内容：<br></span></p>
<ul style="font-size: medium; white-space: normal;  text-align: justify;">
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">需要熟悉掌握 Linux；</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">需要了解一些 Shell 基础，课时中会介绍一些常见的 Shell 命令合集；</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">需要了解 TCP 三次握手原理，这个课时的&nbsp;Shell 命令合集包含对计算机进行网络分析。</span></p></li>
</ul>
<h2 style="white-space: normal; text-align: justify;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">控制台使用技巧</span></p></h2>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">首先，我们来学习控制台的使用技巧，学习掌握控制台的使用技巧后可以帮助我们熟练快速地操作控制台，提高工作效率；还可以通过快捷键方式避免大量的命令输入，减少出错产生的概率。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">那么都有哪些快捷键供我们使用呢，基于我的运维工作经验给你汇总如下：</span></p>
<ul style="font-size: medium; white-space: normal;  text-align: justify;">
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">操作快捷键</span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Ctrl +&nbsp;r：可以快速查找历史命令；</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Ctrl +&nbsp;l：可以清理控制台屏幕；</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Ctrl + a \ Ctrl + e：移动光标到命令行首\行尾；</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Ctrl + w \ Ctrl + k：删除光标之前\之后的内容。</span></p></li>
 </ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">VIM文件编辑快捷键</span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">快捷键ZZ：文件保存并退出。</span></p></li>
 </ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">进程操作快捷键</span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Ctrl + c：强制终止程序的执行；</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Ctrl + z：挂起一个进程；</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Ctrl + d：终端中输入 exit 后回车。</span></p></li>
 </ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">linux命令中快捷键（top）</span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Shift + p：根据 CPU 使用率排序；</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Shift + m：根据内存占用排序。</span></p></li>
 </ul>
</ul>
<h2 style="white-space: normal; text-align: justify;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">Shell 命令合集</span></p></h2>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">熟练掌握相关快捷键后，我们便进入 Shell 命令合集的学习，在这一部分需要你根据课时中所讲的知识在课后进行相关操作的练习，练习过程中希望你可以应用上面所讲的控制台使用技巧。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">需要注意的是这里的 Shell 命令合集不是简单的单一命令使用，而是讲解 Shell 命令组合的使用，它们的适用场景虽然平时可能很少用到，但通过本课时的学习你能够在遇到此类场景时能够得心应手，而不必临时查找或根据经验拼凑，同时期望通过这些组合命令的学习后你对对基础命令的理解可以得到进一步的提升。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">首先，我们需要对 Shell 命令合集做一个分类。</span></p>
<ul style="font-size: medium; white-space: normal;  text-align: justify;">
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">空间分析</span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">场景1：磁盘空间不足，需快速定位日志目录；</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">场景2：系统产生很多碎片文件，导致 inode 资源不足。</span></p></li>
 </ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">指定文件操作</span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">场景1：批量查找文件作内容替换；</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">场景2：批量查找文件作拷贝打包。</span></p></li>
 </ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">链接状态分析</span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">场景：想了解用户请求所建立的网络连接状态分析。</span></p></li>
 </ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">IP 信息提取</span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">场景：shell 脚本中希望快速提取到本机 IP。</span></p></li>
 </ul>
</ul>
<h3 style="white-space: normal; text-align: justify;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">空间分析-场景1</span></p></h3>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">该场景主要应用于当磁盘空间不足，需要快速定位或者对文件使用率进行排序，需要查看哪一些文件目录或者文件占用的空间比较多，就需要如下组合命令。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<pre style="font-size: medium; text-align: justify;">du&nbsp;-x&nbsp;--max-depth=1&nbsp;/&nbsp;|sort&nbsp;-k1&nbsp;-nr</pre>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">我们可以看到这一个命令组合由两个 Shell 命令组成，前面的 du 命令进行磁盘统计，第二个 sort 命令对统计后的数据进行排序，中间通过 | 管道符来传递数据。管道符 | 的作用是将前一个命令的输出传递到下一个命令的输入。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">du 命令中 -x 参数表示跳过其他文件系统，也就是只分析本文件系统里的文件，它可以帮助我们排除一些非本文件系统的统计信息，这样执行速度会更快也不容易出现一些额外的干扰项。--max-depth 参数设置为 1，这样就可以统计出根目录下第一级目录中的所有文件的大小。第二个命令sort中 -k 参数指明具体按照哪一列进行排序，-n 参数表示只对数值进行排序，而 -r 参数表示反向排序，那整体分析sort&nbsp;这一段命令的意思就是指定第一列并按照数据大小做反序排序。</span></p>
<h3 style="white-space: normal; text-align: justify;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">空间分析-场景2</span></p></h3>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">场景 2 适用于系统上产生很多碎片文件时，随之产生大量的 Inode ， Inode&nbsp;用于存放着文件系统中文件的源数据，Inode过渡的使用会导致系统&nbsp;Inode&nbsp;资源不足。这种情况是不正常的，这个时候分析如果通过du 命令指能具体展示出磁盘空间的使用情况，但并不能分析出具体目录下产生了多少碎片文件，我们就需要如下的命令组合来对文件进行统计分析。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<pre style="font-size: medium; text-align: justify;">find&nbsp;-type&nbsp;f|awk&nbsp;-F/-v&nbsp;OFS=/'{$NF="";dir[$0]++}END{for(i&nbsp;in&nbsp;dir)print&nbsp;dir[i]""i}'|sort&nbsp;-k1&nbsp;-nr|head</pre>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">基于管道可以将这个命令组合切割成四部分，分别是 find、awk、sort、head 命令。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 22px; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">其中 find 命令通过 -type -f 参数查找指定文件类型的文件，然后将查找结果通过管道传递给 awk，它可以把文本内容按行进行格式化输出并展示，-F /&nbsp;指定处理文件时字符串之间以 / 进行分割，-v&nbsp;OFS=/ 表示文件显示结果时以 / 进行分割展示。对于awk命令整体规则而言有一个 {} END {} 格式，前面的 {} 表示行处理操作，END{} 表示行处理后需要进行整体结果出。在行处理操作逻辑中，设置$NF&nbsp;为空表示将每一行的文件名信息去除，从而只保留目录路径，dir 是一个自增数组，用于统计结果。最后通过 for 循环进行遍历输出dir关联数组中所有行信息。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 22px; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">因为这个命令组合比较复杂，我们在控制台中来看具体的演示，首先在控制台中输入这一串命令组合。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 22px; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><img src="https://s0.lgstatic.com/i/image3/M01/65/4A/CgpOIF5BIhKADtFGAANv__aeR0g437.png"></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">你可以看到在当前目录路径通过执行命令，结果中已经把每一个产生文件的路径都展示出来了，并且前面还会显示在每一个路径下一共包含了多少文件，如果我们系统&nbsp;提示inode&nbsp;使用率问题，需要分析出哪个路径下的文件数最多，这时就可以通过&nbsp;这个组合命令来进行分析。</span></p>
<h3 style="white-space: normal; text-align: justify;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">文件操作-场景1</span></p></h3>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">文件操作的场景主要有两个，第一个场是批量文件内容需要进行替换，也就是当我们在一个文件目录下面有多级子目录，并且子目录中有大量的文件，而我们需要对目录下的某一个名称的文件批量的查找替换内容。面对这种场景我们可以使用如下的组合命令：</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<pre style="font-size: medium; text-align: justify;">find&nbsp;./-type&nbsp;f&nbsp;-name&nbsp;consumer.xml&nbsp;-exec&nbsp;sed&nbsp;-i"s/aaaaaa/bbbbbb/g"{}\;</pre>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">我们通过&nbsp;find +&nbsp;路径&nbsp;的方式查找需要批量修改的指定的文件名，比如命令中的 consumer.xml 文件，查找到文件后通过 find 自带的参数 exec 将结果传递给另外一条命令 sed 来进行下一步命令的处理。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">find 命令中，-name 参数指定查找的文件名，-exec 参数将查找到的内容传递给下一个命令去继续执行相关逻辑，sed 命令主要对文件内容进行替换，这里会将 consumer 文件中的 aaaaaa 替换成 bbbbbb，这就是一个批量查找替换的操作。</span></p>
<h3 style="white-space: normal; text-align: justify;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">文件操作-场景2</span></p></h3>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">文件操作的第二种场景是我们需要对文件进行批量的打包、拷贝，你可以看到下面这样的一个组合命令：</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<pre style="font-size: medium; text-align: justify;">(find&nbsp;.&nbsp;-name&nbsp;"*.txt"|xargs&nbsp;tar&nbsp;-cvf&nbsp;test.tar)&nbsp;&amp;&amp;&nbsp;cp&nbsp;-f&nbsp;test.tar&nbsp;/home/.</pre>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">首先我们来看下括号中的部分，括号中包含两条命令，它们使用管道符进行连接，括号外通过"&amp;&amp;"符号与第三条命令进行连接，也就是我们首先需要执行括号中的组合命令，先查找所有 .txt 文件，然后将结果传递给 xargs 命令进行打包，如果打包成功后才将压缩包传递给 cp 命令进行拷贝。</span></p>
<h3 style="white-space: normal; text-align: justify;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">网络连接状态分析</span></p></h3>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">对于网络连接状态分析是运维工程师经常需要做的事情，因为我们经常需要了解系统对外提供的网络服务是否正常，并了解它们的连接状态，这时就可以通过如下的命令组合进行操作：</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<pre style="font-size: medium; text-align: justify;">netstat&nbsp;-n&nbsp;|&nbsp;awk&nbsp;'/^tcp/&nbsp;{++S[$NF]}&nbsp;END&nbsp;{for(a&nbsp;in&nbsp;S)&nbsp;print&nbsp;a,&nbsp;S[a]}'</pre>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">我们先来分析下这个命令组合的构成，整体上来说它由两个命令构成，第一个命令是 netstat -n，这个命令负责查看主机上的所有 TCP、UDP&nbsp;连接信息，而 awk 命令则负责对这些信息进行进一步的处理，awk 后有一个用两个&nbsp;"斜杠"&nbsp;括起来的正则表达式，主要用来匹配以 tcp 开头的每一行信息，所以这里的正则表达式起到了一个过滤的作用（只分析tcp的连接），后面则是对信息过滤后进行具体的统计和输出。</span></p>
<h3 style="white-space: normal; text-align: justify;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">IP信息提取</span></p></h3>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">而另外一个场景就是提取主机上的 IP 信息，这里推荐使用如下的命令组合：</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<pre style="font-size: medium; text-align: justify;">ip&nbsp;a|grep&nbsp;"global"|awk'{print&nbsp;$2}'|awk&nbsp;-F/'{print&nbsp;$1}'</pre>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">可以看到它的结构组成也比较简单，分别是由四个命令组合而成，前面的 ip a 负责查看主机上所有网卡的信息，然后通过 grep 进行条件过滤，再通过 awk&nbsp;实现第二列内容输出，最后通过 awk 以指定 / 作为分隔符来打印第一列的信息。</span></p>
<h2 style="white-space: normal; text-align: justify;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">常见问题答疑</span></p></h2>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">最后，分享一下&nbsp;对Shell&nbsp;常有的几个疑问。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">问题一：Shell 适不适合作多并发任务？</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">答案：不适合，在 Shell 中一般需要通过 nohup 方式将需要并发执行的命令放入后台，但这样操作存在一些问题，包括：</span></p>
<ul style="font-size: medium; white-space: normal;  text-align: justify;">
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">进程的状态不好控制；</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">进程间信息共享一般以文件方式。</span></p></li>
</ul>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">等等，</span><span style="background-color: rgb(255, 255, 255); color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">所以，我们当需要进行大的自动化工程任务</span><span style="background-color: rgb(255, 255, 255); color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">需要作并发任务</span><span style="background-color: rgb(255, 255, 255); color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">时，建议选择 Python、Go、PHP 等语言。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">问题二：Shell 的远程执行命令方式是什么？</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">答案：当 Shell 进行远程执行命令时，通常通过 ssh xx@xxx.xxx.xxx.xxx&nbsp;/home/ieson/imoocc.sh&nbsp;参数的方式，但如果是批量主机任务，建议选择 ansible、saltstack 这样成熟且专业的工具实现。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">问题三：Shell 适合用在什么场景中？</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);"><br></span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">答案：Shell 适合用在追求运维高效（非性能高效）要求的简单场景中，如日志切割、进程分析、系统初始化等。</span></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="white-space: normal; margin-top: 0pt; margin-bottom: 0pt; text-align: justify; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">本专栏课中的所有案例配置及源代码，可以课后通过</span><a class="ql-link ql-author-25848023" href="http://www.jesonc.com/jeson/2020/02/07/ywgs36/" target="_blank" rel="noopener noreferrer nofollow" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">http://www.jesonc.com/jeson/2020/02/07/ywgs36/</a><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; background-color: rgb(255, 255, 255);">自己下载，密码为 mukelaoshi。</span></p>

---

### 精选评论

##### *喜：
> 精简实用，满满干货

##### **涵：
> 结合场景说明，很干活

##### **泰：
> 要多多练

##### *誉：
> <div>复制上面的命令，请注意格式：</div><div>&nbsp;find /root/ -type f -name consumer.xml -exec sed -i "s/bbbb/cccc/g" {} \;</div><div><br></div>

##### *迈：
> 赞，很实用的技能

##### **卡拉：
> 文件操作的第二种场景是我们需要对文件进行批量的打包

