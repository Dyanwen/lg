<p style="text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">本课时我们主要讲解 Selenium 的隐式等待与显式等待，并了解隐式等待和显式等待的主要用途。</span><br></p>
<h2><p><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">等待类型</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">首先，显式等待与隐式等待都属于 Selenium 的等待机制，我们常用的 Selenium 等待机制主要可以分为 3 类。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">第一类是隐式等待，隐式等待表示当一个元素没有出现的时候需要轮询等待的默认最长时间，如果没有设置隐式等待，当第一次查找控件的时候如果发现控件不存在就会直接报错，这个时候会导致你的 case 不稳定。比如在网速不好的情况下，控件出现往往会有延迟。也包括有些控件是动态出现的，出现前需要一定的时间，这样往往会导致报错，所以通常情况下我们会在 case 中添加一个隐式等待，以确保我们的 case 可以更加稳定顺畅的执行。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">第二类是显式等待，显式等待通常用于解决隐式等待不足以解决的问题场景，比如说有的时候元素本身已经存在但是我想捕捉的是元素的状态切换，比如可点击或可见状态，这个时候控件已经存在，如果不利用显式等待就不是很好处理这样的需求，这个时候我们就需要使用显式等待，它可以用来捕捉元素的转态改变。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">第三类是死等，顾名思义，死等就是一直等待，死等对于新手比较友好，比如我发现某个控件没有出现，就通过 sleep 等待 5 秒钟，这样 sleep 就会让控件强行进入等待，显然这样会拖长我们的测试用例运行时间，所以通常情况下不建议你去使用死等。</span></p>
<h2><p><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">显式等待与隐式等待的区别</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">接下来，我们看下显示等待与隐式等待有什么区别，通常情况下，我们会默认设置一个隐式等待的时间，这个需要根据你的应用的实际情况而定，如果我们不主动设置则默认设置为 0，也就是第一次运行如果找不到控件就直接报错。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">一般情况，这个默认等待时间会设置的非常短，比如 5~6 秒，一般不会超过 10 秒，然后在这个等待时间间隔内，如果元素没有出现则以每隔 0.5 秒的间隔进行重试，一直到找到为止，如果最后找不到就会超时报错，一般隐式等待只能用于 find_element，它无法用于复杂的等待。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">所以这个时候就需要显式等待进行有效的补充了，显式等待可以解决上传问题，比如上传时需要等待 20~30 秒，在这个过程中会一直有一个 loading 框，我们需要等待这个 loading 框消失，所以这时我们会编写显式等待以处理这种等待时间太长的场景。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">显式等待还可以处理控件状态改变的问题，比如消失、可点击等比较灵活的状态切换。</span></p>
<h2><p><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">显式等待与隐式等待的使用</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">接下来，我们具体看下这两个状态怎么使用，首先我们返回到上一课时所录用的测试用例。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><br></span></p>
<pre>self.driver.implicitly_wait(5)</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">我们来看具体代码，代码中我们首先打开了一个网址，然后点击了某个按钮，在点击按钮的过程中可能会因为控件没有加载完成而报错，通常情况下我们会在 driver 初始化完成之后设置一个隐式等待，并给它设置一个时间，比如这里等待 5 秒钟。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><br></span></p>
<pre>self.wait=WebDriverWait(self.driver,30)</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"></span><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">那么，显式等待怎么用呢</span>，比如我们输入完成之后需要等待某个元素的状态发生变化，这个时候就可以引入 WebDriverWait，WebDriverWait 传入的第一个参数是 self.driver，然后给定一个最大超时时间，比如这里设置为 30 秒。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">然后初始化这样的一个wait实例，通常情况下我们会把 wait 放在 driver 初始化之后，这样就设置了一个显式等待，以方便后面进行调用。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><br></span></p>
<pre>self.wait.until(expecter_conditions.presence_of_element_located(element))</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"></span><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">我们可以看到 wait 主要有两个方法，一个是 until，另一个是 until_not，这两个方法的执行结果正好相反，我们看下最常用的 until 方法，until 方法表示等待控件状态是否发生切换，这个状态我们可以使用 &nbsp;expected.condition 包下面的方法进行检测。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">&nbsp;</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">你可以看到这些方法中包含检测元素出现的方法</span>，presence 表示这个控件是否存在在界面结构中但它不一定可见，它只要存在就认为状态是没有问题的。我们使用 presence_of_element方法 ，传入一个控件定位符。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><br></span></p>
<pre>self.wait.until(expected_condiitions.visibility_of_element_located(element))</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"></span></span><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">visibility_of_element_located方法是</span>解决的情况是除了需要元素存在于界面结构之中还要求元素是可见的，针对可见状态的检测我们使用 visibility 方法，然后再把 element传进来就可以了。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><br></span></p>
<pre>slefe.wait.until(expected_conditions.element_to_be_clickable(element))</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"></span><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">element_to_be_clickable方法等待一个元素可以被点击</span>，因为元素出现并不一定代表可以被点击，有的时候因为加载的延迟导致控件出现而点击没有效果，我们需要等待一段时间之后才可以点击成功，对于这种情况如果点击太早则没有效果，后面的测试流程就一定会报错。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><br></span></p>
<pre>self.wait.until(lambda&nbsp;x:"title"&nbsp;in&nbsp;self.driver.page_source)</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"></span><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">除了这些状态之外，self.wait 还支持非常灵活的方法，</span>比如你可以自己写一个 until 方法，这个方法可以是命名方法也可以是匿名方法。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">&nbsp;</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">匿名方法使用 lambda，在 lambda 表达式中你可以指定一个测试条件，比如当前页面必须包含某个特定元素，你也可以将 lambda 表达式换成一个命名函数。</span><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>time.sleep(5)</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">最后是死等，除了一些特殊的场景外不建议你使用这个方法。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">我们可以看出隐式等待是一定需要配置的，可以解决大多数的等待问题，个别复杂的等待可以通过显式等待解决，死等不建议使用。</span></p>

---

### 精选评论


