<p style="line-height: 1.75em; text-align: justify;"><span></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本课时我们进入 Selenium 常用自动化 API 的学习，我们先来看下 Selenium 有哪几种常用的 API。</span></p> 
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Web 控件定位与常见操作</span></p></h2> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">首先，我们从 Web 控件定位与常见操作开始讲起，我们先来看下 Selenium 常见的定位方法，其实在上一课时录制 case 的时候我们就看到了 Selenium 有一些定位， Selenium 总共有多少种定位方法呢，这里给你做一次梳理，Selenium 主要的定位方法分别是：</span></p> 
<ul> 
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">ID 定位，根据控件的 ID 进行定位；</span></p></li> 
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">NAME 定位，根据控件的 NAME 属性进行定位；</span></p></li> 
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">TAG NAME 定位，根据控件的 TAG 标签进行定位；</span></p></li> 
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Class NAME 定位，根据控件的 Class 属性进行定位；</span></p></li> 
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">CSS定位</span></p></li> 
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">LinkText定位</span></p></li> 
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">XPath定位</span></p></li> 
</ul> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">前 4 种是封装好的使用快捷的定位方法，这些定位本质上底层都是使用 CSS 定位符，关于 CSS 定位符，Selenium 也给我们提供了一个独特的 API 叫作 find_element_by_css_selector，我们可以从 self.driver 方法中找到 find 系列方法集。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">还有一种定位方式比较特殊，它不再使用 CSS 定位，而是采用更底层的 XPath 定位，因为 HTML 本身也是一种特殊的 XML 文件，XML 也支持 XPath，所以我们也可以使用 XPath 来对控件进行定位。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">除此之外，还有一种是 Link 定位，Link 定位通过文本文件的标签内容进行定位，本质上它也可以写成 XPath 定位风格。总体上 Selenium 给我们提供的定位 API 就以上这几种，那我们接下来看具体怎么写这些定位符。</span></p> 
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">定位符的正确写法</span></p></h3> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">关于定位符的正确写法，我们可以看到传统的方法 Selenium 的 driver 中已经给我们封装了 driver.find_element_by_name 系列方法，这个系列方法在实际工作中并不建议你使用，因为很多时候我们都需要做进一步的封装，或者控件的 ID 会发生变化，这个时候如果采用find_element_by_name 系列方法，当 ID 发生变化时就意味着 case 的 name </span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">定位</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">也需要发生变化，这样</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">不利于</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">复用</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">与解耦</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">，所以不建议你使用这种方法。</span><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">通常情况下，建议你使用第 2 种或第 3 种方法，第 2 种和第 3 种方法会把一个控件的定位符描述好，然后把它存入一个变量中，这样就可以通过引用变量进行定位。在平时写代码时我们也会尽量使用 find_element 方法， 然后在方法中设置我们需要定位控件的定位符。</span></p> 
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">CSS 定位</span></p></h3> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">关于 Selenium 更复杂的定位多数情况下是使用 CSS，通常有些控件是没有 name 的，或者没有 ID，这种情况就需要使用 class 进行定位，而 class 定位又有非常多的属性，这些属性有时需要结合使用，比如说需要使用 ID 的同时，还需要结合 class属性，甚至还有父子关系，这样定位才会更精准，在这种复杂的情况下默认的几种方法就不太可行了。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这个时候我们需要使用 CSS 的高级定位，比如父子关系的兄弟节点，我们都可以使用 CSS 高级定位查找到，CSS 在 H5 开发中也用的非常的多，所以在 Selenium 中我更推荐你使用 CSS 定位，关于CSS定位的更多细节可以参考 &nbsp;</span><a class="ql-link ql-author-6398505" href="https://www.w3schools.com/cssref/css_selectors.asp" target="_blank" rel="noopener noreferrer nofollow" style="text-decoration: underline; color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">https://www.w3schools.com/cssref/css_selectors.asp</span></a></p> 
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">XPath 定位</span></p></h3> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">XPath 定位在 Web 测试中使用的比较少，而在移动端的测试中使用的比较多，不过在某些特殊场景下，XPath 也可以帮助你解决一些 Web 测试中的问题，关于 XPath 定位你可以参考 https://www.w3schools.com/xml/xpath_syntax.asp 文档。</span></p> 
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">控件对应的操作方法</span></p></h2> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">了解了控件的定位之后，接下来我们再来了解控件对应的操作方法，在前面的演练中你也看到了，通常是包含 click 和 send_keys 两个关键操作的，除了这两个操作之外呢，我们还需要获取到控件的关键属性来进行后续的断言，这个时候我们需要使用 get_attribute，所以这 3 种方法就组成了我们对控件进行相关操作的主流方法。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">接下来我们通过测试一个新的网站来看下典型的测试用例该如何编写，我们进入 PyCharm，回到之前的项目中去，这次我们使用一个新的 case，打开这个网站的地址，来模拟一个新网站的搜索操作，我们点击搜索框输入Selenium 并回车，然后就可以得到搜索结果，我们在搜索结果中断言是否包含 Selenium，接下来就以这个 case 来教你怎么去编写具体的代码。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<pre>def&nbsp;test_testingstudio(self):
&nbsp;&nbsp;&nbsp;&nbsp;self.driver.get("https://testing-studio.com/")
&nbsp;&nbsp;&nbsp;&nbsp;self.driver.find_element(By.CSS_SELECTOR,'search-button').click()
&nbsp;&nbsp;&nbsp;&nbsp;self.driver.find_element(By.CSS_SELESTOR,'search-trem').send_keys("selenium")</pre> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></span><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">首先，我们需要复制网址并把它写进来，在这里使用 self.driver.get，接下来我们使用 find.element 去查看点击的搜索控件，使用 Chrome 给我们提供的便捷工具就可以找到搜索控件，然后右键复制它的 selector，这样就可以把 CSS 定位符直接拿到了。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">拿到定位符之后我们使用 BY.CSS 定位，然后使用对应定位符定位并执行 click 方法，接下来我们再定位搜索的输入框并使用 send_keys 方法。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<pre>self.driver.find_element(By.CSS_SELECTOR,'#search-term')</pre> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></span><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">输入完成之后就会进入搜索结果页面，在结果页中我们想断言第一个搜索结果的标题中是否匹配我的关键字，这个时候就可以继续使用检查找到它的 selector，我们继续使用 CSS 定位符，然后需要断言它的内容，我们看下它的定位符“ember377”，很明显定位符里面包含数字，如果下次跑 case 时数字变化了，case 就会失败，通常情况下如果我们发现定位符写的不够精准就需要去找到正确的稳定的定位符。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/6D/3D/Cgq2xl5c9UOARr4jAAG5zhiZE-8151.png"></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们可以看到它的 class 属性的 selector 是不变的，所以我们把它复制过来作为正式的定位符，因为这是一个class属性，class在css定位符里是用一个点表示的，点表示控件的类属性，如果是 ID 属性我们就需要使用 # 号，CSS 是有自己的定位语法的，它非常的灵活，然后我们使用 get_attribute。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<pre>def&nbsp;test_testingstudio(self):
&nbsp;&nbsp;&nbsp;&nbsp;self.driver.get("https://testing-studio.com/")
&nbsp;&nbsp;&nbsp;&nbsp;self.driver.find_element(By.CSS_SELECTOR,'search-button').click()
&nbsp;&nbsp;&nbsp;&nbsp;self.driver.find_element(By.CSS_SELESTOR,'search-trem').send_keys("selenium")
&nbsp;&nbsp;&nbsp;&nbsp;assert&nbsp;"Selenium"&nbsp;in&nbsp;self.driver.find_element(By.CSS_SELESTOR,
'.topic-title').text</pre> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">get_attribute 可以获取到你想要的控件的属性，因为我们只涉及文本，所以可以使用它的一个简单的方法 .text，它会获取到控件的文本内容，获取到文本内容之后断言它里面是否包含 Selenium，所以我们使用 assert，然后判断 Selenium 是否在文本内，这样整个 case 就编写完了。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<pre>def&nbsp;test_testingstudio(self):
&nbsp;&nbsp;&nbsp;&nbsp;self.driver.get("https://testing-studio.com/")
&nbsp;&nbsp;&nbsp;&nbsp;self.driver.find_element(By.CSS_SELECTOR,'search-button').click()
&nbsp;&nbsp;&nbsp;&nbsp;input_element=self.driver.find_element(By.CSS_SELESTOR,'search-trem')
&nbsp;&nbsp;&nbsp;&nbsp;input_element.send_keys("selenium")
&nbsp;&nbsp;&nbsp;&nbsp;input_element.send_keys(Keys.ENTER)
&nbsp;&nbsp;&nbsp;&nbsp;assert&nbsp;"Selenium"&nbsp;in&nbsp;self.driver.find_element(By.CSS_SELESTOR,
'.topic-title').text</pre> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这个时候的运行结果是少一个按回车的操作的，我们回到代码中，一旦需要多次用到同一个定位符的时候，就可以把获取到的控件保存下来，使用 send_keys，send_keys 中我们选择 Keys.ENTER，然后我们再运行就没有任何问题了，这样 case 就编写完成了，通过这个 case 我们了解到对于任何新的网站都可以快速的进行一个测试，当然这也是最简单的一个案例。</span></p> 
<ul> 
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Actions</span></p></li> 
</ul> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">接下来，我们看下除了上面讲到的常见的 3 种方法外还有哪些常见的 API，比如有的时候我需要滑动操作，把一个控件滑动到另外一个控件内，你可以使用 Actions 进行操作，如果涉及移动端还有 TouchAction，因为篇幅原因具体的用法就不再进行讲解了，你可以课后自己看下对应的文档。</span></p> 
<ul> 
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">多窗口处理与 frame 切换</span></p></li> 
</ul> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">再一个是多窗口切换，我们点开一个控件出现一个新窗口的时候，也需要检测切换，如果不检测切换就算你的控件存在也无法检测到，在这种情况下我们需要使用 window_handles、switch_to.window、switch_to.frame 这 3 个 API。window_handles 可以获取目前浏览器中有几个窗口，switch_to 可以切换到对应的窗口或 frame 里，还有更多的 API 你可以课后查看 self.driver的具体方法的官方文档，这里就不再具体讲解了。</span></p> 
<p></p>

---

### 精选评论

##### **东：
> 定位都用xpath不就好了么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; selenium更推荐的使用css，而不是xpath。css的定位更精准，更好用。而且更易于维护。

