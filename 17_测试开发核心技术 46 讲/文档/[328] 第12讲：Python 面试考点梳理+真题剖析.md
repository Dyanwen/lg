<p style="text-align: justify; line-height: 1.75em;"></p>
<p style="text-align:justify;line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本课时我们主要梳理 Python 相关的面试考点，如果你是一个新人，去一家公司面试测试工程师时，除了考察基础的 Shell 知识点外，还会考察你的基础编程能力，这个时候 Python 面试考点就是你必须要过得第一道关，接下来我们就一起看下 Python 面试题中都包括哪些内容，在这里做一个结构化的梳理。</span><br></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Python 基础</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">首先，关于 Python 基础知识我们在面试中会经常遇到这两大类问题，第一类是语法知识，第二类是基本数据类型。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在语法知识中，面试官会考察你对基本语言的掌握程度，这里会问你一些最经常用到的编程语法，比如 super、self、cls 等如何使用，以及异常处理、循环相关的语法知识，主要考察你在实际工作中有没有具体用过这些语法。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第二类是基本数据结构，在基本数据结构中会考察你对前面课时讲过的 Python 中的基础数据类型的掌握程度，以及围绕这些数据结构所做的算法等，比如列表和字典有什么区别？元祖和列表有什么区别等？</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这里重点剖析下关于列表处理的面试真题：</span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">第一道题是如何从一个带有重复项的列表中得出一个合并重复项的列表。解题思路是这样的，首先将带重复项的列表处理为一个不重复数据的集合，然后再将集合转化为列表。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">第二道题考察的也是基础知识，首先对列表使用切片，切片的下标是从 10 开始一直到</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">切片默认值</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">，但是因为列表只有 5 个元素，所以使用切片的结果为空。</span><br></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">第三个问题是如何将列表中的数字合并转成字符串，因为它需要拼接到一起，这时我们首先想到 join 操作，而列表中的数据本身是数字，我们先将它转化成字符格式，然后使用 join 方法拼接成完整的字符串。</span><br></p></li>
</ul>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这里的知识比较简单，就不再带领你进行演练了，课后你可以自己进行练习，如果你不知道标准答案，可以在课程下留言，我会将答案公布到留言中。</span><br></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">数据结构与算法</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">然后，第二个重点考察的知识点是数据结构与算法，比如常见的数据结构列表、词典，或是在此基础上的扩展数据结构，比如堆栈、队列、链表。这里需要注意 Python 语法的二叉树实现，我们需要掌握如何使用 Python 实现二叉树遍历。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在数据结构与算法中还会考察你对基础数据的分析和统计能力，比如有一个几十万行的日志，我们如何从中统计出错的日志，以及它的基本数据情况是什么的，这里涉及一系列的数据查找汇总知识，具体的内容我们会在数据结构与算法课时进行详细讲解。</span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">领域知识</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">最后，我们一起来看下领域知识的考察点，在领域知识中你需要掌握系统编程、网络编程、自动化测试、数据编程、测试用例、Web 开发等内容。</span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">系统编程中会考察你对进程、线程知识的了解程度，比如如何开启关闭线程，如何并发等等；</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">网络编程在我们的工作中也是非常重要的，这里主要考察 TCP socket、HTTP 协议请求等内容。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">自动化测试中主要考察 UI 自动化测试和接口自动化测试。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">数据编程主要考察 XML 和 JSON 结构的处理，这里需要重点掌握 JSON 数据结构。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">还有就是测试用例和 Web 开发，测试用例中需要能够掌握 unittest、pytest 等测试框架，以及在此基础上衍生出的其他第三方框架。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Web 开发中需要你对 Flask、HTML、JavaScript、Vue 有一定的了解。如果你能够掌握了上述这些内容在面试中一定会是加分项。</span></p></li>
</ul>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在 Python 面试的相关问题中首先会考察你对 Python 基础知识的掌握程度，其次是对数据结构和算法的了解，最后考察你对领域专业知识的掌握程度，如果你发现自己还对很多知识的掌握有所欠缺，希望你课后能够复习相关的知识，有些内容我们也会在接下来的课程中详细讲解。</span></p>

---

### 精选评论

##### **7408：
> 1.b=list(set(a))2.Null3."".join([str(i) for i in a])

##### *翠：
> <div>答案可以分享一下么</div><div><br></div>

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 可以关注 拉勾教育 公众号 咨询小助手

