<p data-nodeid="108788" class="">上个课时我们讲解了 Ajax 的分析方法，利用 Ajax 接口我们可以非常方便地完成数据的爬取。只要我们能找到 Ajax 接口的规律，就可以通过某些参数构造出对应的的请求，数据自然就能被轻松爬取到。</p>
<p data-nodeid="110453" class="">但是，在很多情况下，Ajax 请求的接口通常会包含加密的参数，如 token、sign 等，如：<a href="https://dynamic2.scrape.center/" data-nodeid="110457">https://dynamic2.scrape.center/</a>，它的 Ajax 接口是包含一个 token 参数的，如图所示。</p>


<p data-nodeid="108790"><img src="https://s0.lgstatic.com/i/image3/M01/7D/12/Cgq2xl59oBaAXsW1AADPHO1rr_g541.png" alt="" data-nodeid="108983"></p>
<p data-nodeid="108791">由于接口的请求加上了 token 参数，如果不深入分析并找到 token 的构造逻辑，我们是难以直接模拟这些 Ajax 请求的。</p>
<p data-nodeid="108792">此时解决方法通常有两种，一种是深挖其中的逻辑，把其中 token 的构造逻辑完全找出来，再用 Python 复现，构造 Ajax 请求；另外一种方法就是直接通过模拟浏览器的方式，绕过这个过程。因为在浏览器里面我们是可以看到这个数据的，如果能直接把看到的数据爬取下来，当然也就能获取对应的信息了。</p>
<p data-nodeid="108793">由于第 1 种方法难度较高，在这里我们就先介绍第 2 种方法，模拟浏览器爬取。</p>
<p data-nodeid="108794">这里使用的工具为 Selenium，我们先来了解一下 Selenium 的基本使用方法吧。</p>
<p data-nodeid="108795">Selenium 是一个自动化测试工具，利用它可以驱动浏览器执行特定的动作，如点击、下拉等操作，同时还可以获取浏览器当前呈现的页面源代码，做到可见即可爬。对于一些使用 JavaScript 动态渲染的页面来说，此种抓取方式非常有效。本课时就让我们来感受一下它的强大之处吧。</p>
<h3 data-nodeid="108796">准备工作</h3>
<p data-nodeid="108797">本课时以 Chrome 为例来讲解 Selenium 的用法。在开始之前，请确保已经正确安装好了 Chrome 浏览器并配置好了 ChromeDriver。另外，还需要正确安装好 Python 的 Selenium 库。</p>
<p data-nodeid="108798">安装过程可以参考：<a href="https://cuiqingcai.com/5135.html" data-nodeid="108994">https://cuiqingcai.com/5135.html</a>&nbsp;和&nbsp;<a href="https://cuiqingcai.com/5141.html" data-nodeid="108998">https://cuiqingcai.com/5141.html</a>。</p>
<h3 data-nodeid="108799">基本使用</h3>
<p data-nodeid="108800">准备工作做好之后，首先来看一下 Selenium 有一些怎样的功能。示例如下：</p>
<pre class="lang-python" data-nodeid="108801"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.common.by&nbsp;<span class="hljs-keyword">import</span>&nbsp;By&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.common.keys&nbsp;<span class="hljs-keyword">import</span>&nbsp;Keys&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.support&nbsp;<span class="hljs-keyword">import</span>&nbsp;expected_conditions&nbsp;<span class="hljs-keyword">as</span>&nbsp;EC&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.support.wait&nbsp;<span class="hljs-keyword">import</span>&nbsp;WebDriverWait
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
<span class="hljs-keyword">try</span>:
browser.get(<span class="hljs-string">'https://www.baidu.com'</span>)
input&nbsp;=&nbsp;browser.find_element_by_id(<span class="hljs-string">'kw'</span>)
input.send_keys(<span class="hljs-string">'Python'</span>)
input.send_keys(Keys.ENTER)
wait&nbsp;=&nbsp;WebDriverWait(browser,&nbsp;<span class="hljs-number">10</span>)
wait.until(EC.presence_of_element_located((By.ID,&nbsp;<span class="hljs-string">'content_left'</span>)))
print(browser.current_url)
print(browser.get_cookies())
print(browser.page_source)&nbsp;
<span class="hljs-keyword">finally</span>:
browser.close()
</code></pre>
<p data-nodeid="108802">运行代码后会自动弹出一个 Chrome 浏览器，浏览器会跳转到百度，然后在搜索框中输入 Python，接着跳转到搜索结果页，如图所示。</p>
<p data-nodeid="108803"><img src="https://s0.lgstatic.com/i/image3/M01/03/FC/Ciqah159oBaACo6HAAyBbpN_drw824.png" alt="" data-nodeid="109004"></p>
<p data-nodeid="108804">此时在控制台的输出结果如下：</p>
<pre class="lang-html" data-nodeid="108805"><code data-language="html">https://www.baidu.com/s?ie=utf-8&amp;f=8&amp;rsv_bp=0&amp;rsv_idx=1&amp;tn=baidu&amp;wd=Python&amp;rsv_pq=&nbsp;
c94d0df9000a72d0&amp;rsv_t=07099xvun1ZmC0bf6eQvygJ43IUTTUOl5FCJVPgwG2YREs70GplJjH2F%2BC
Q&amp;rqlang=cn&amp;rsv_enter=1&amp;rsv_sug3=6&amp;rsv_sug2=0&amp;inputT=87&amp;rsv_sug4=87&nbsp;
[{'secure':&nbsp;False,
&nbsp;'value':&nbsp;'B490B5EBF6F3CD402E515D22BCDA1598',&nbsp;
&nbsp;'domain':&nbsp;'.baidu.com',&nbsp;
&nbsp;'path':&nbsp;'/',
&nbsp;'httpOnly':&nbsp;False,&nbsp;
&nbsp;'name':&nbsp;'BDORZ',&nbsp;
&nbsp;'expiry':&nbsp;1491688071.707553},&nbsp;
&nbsp;
&nbsp;{'secure':&nbsp;False,&nbsp;
&nbsp;'value':&nbsp;'22473_1441_21084_17001',&nbsp;
&nbsp;'domain':&nbsp;'.baidu.com',&nbsp;
&nbsp;'path':&nbsp;'/',
&nbsp;'httpOnly':&nbsp;False,&nbsp;
&nbsp;'name':&nbsp;'H_PS_PSSID'},&nbsp;

&nbsp;{'secure':&nbsp;False,&nbsp;
&nbsp;'value':&nbsp;'12883875381399993259_00_0_I_R_2_0303_C02F_N_I_I_0',&nbsp;
&nbsp;'domain':&nbsp;'.www.baidu.com',
&nbsp;'path':&nbsp;'/',&nbsp;
&nbsp;'httpOnly':&nbsp;False,&nbsp;
&nbsp;'name':&nbsp;'__bsi',&nbsp;
&nbsp;'expiry':&nbsp;1491601676.69722}]
<span class="hljs-meta">&lt;!DOCTYPE&nbsp;<span class="hljs-meta-keyword">html</span>&gt;</span>
<span class="hljs-comment">&lt;!--STATUS&nbsp;OK--&gt;</span>...
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span>
</code></pre>
<p data-nodeid="108806">源代码过长，在此省略。可以看到，当前我们得到的 URL、Cookies 和源代码都是浏览器中的真实内容。</p>
<p data-nodeid="108807">所以说，如果用 Selenium 来驱动浏览器加载网页的话，就可以直接拿到 JavaScript 渲染的结果了，不用担心使用的是什么加密系统。</p>
<p data-nodeid="108808">下面来详细了解一下 Selenium 的用法。</p>
<h3 data-nodeid="108809">声明浏览器对象</h3>
<p data-nodeid="108810">Selenium 支持非常多的浏览器，如 Chrome、Firefox、Edge 等，还有 Android、BlackBerry 等手机端的浏览器。</p>
<p data-nodeid="108811">此外，我们可以用如下方式进行初始化：</p>
<pre class="lang-python" data-nodeid="108812"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser&nbsp;=&nbsp;webdriver.Firefox()&nbsp;
browser&nbsp;=&nbsp;webdriver.Edge()&nbsp;
browser&nbsp;=&nbsp;webdriver.Safari()
</code></pre>
<p data-nodeid="108813">这样就完成了浏览器对象的初始化并将其赋值为 browser 对象。接下来，我们要做的就是调用 browser 对象，让其执行各个动作以模拟浏览器操作。</p>
<h3 data-nodeid="108814">访问页面</h3>
<p data-nodeid="108815">我们可以用 get 方法来请求网页，只需要把参数传入链接 URL 即可。比如，这里用 get 方法访问淘宝，然后打印出源代码，代码如下：</p>
<pre class="lang-python" data-nodeid="108816"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.taobao.com'</span>)&nbsp;
print(browser.page_source)&nbsp;
browser.close()
</code></pre>
<p data-nodeid="108817">运行后会弹出 Chrome 浏览器并且自动访问淘宝，然后控制台会输出淘宝页面的源代码，随后浏览器关闭。</p>
<p data-nodeid="108818">通过这几行简单的代码，我们就可以驱动浏览器并获取网页源码，非常便捷。</p>
<h3 data-nodeid="108819">查找节点</h3>
<p data-nodeid="108820">Selenium 可以驱动浏览器完成各种操作，比如填充表单、模拟点击等。举个例子，当我们想要完成向某个输入框输入文字的操作时，首先需要知道这个输入框在哪，而 Selenium 提供了一系列查找节点的方法，我们可以用这些方法来获取想要的节点，以便执行下一步动作或者提取信息。</p>
<p data-nodeid="108821"><strong data-nodeid="109022">单个节点</strong></p>
<p data-nodeid="108822">当我们想要从淘宝页面中提取搜索框这个节点，首先要观察它的源代码，如图所示。</p>
<p data-nodeid="108823"><img src="https://s0.lgstatic.com/i/image3/M01/03/FC/Ciqah159oBaAN2vyAAKaGSF3EU4066.png" alt="" data-nodeid="109025"></p>
<p data-nodeid="108824">可以发现，它的 id 是 q，name 也是 q，此外还有许多其他属性。此时我们就可以用多种方式获取它了。比如，find_element_by_name 代表根据 name 值获取，find_element_by_id 则是根据 id 获取，另外，还有根据 XPath、CSS 选择器等获取的方式。</p>
<p data-nodeid="108825">我们用代码实现一下：</p>
<pre class="lang-python" data-nodeid="108826"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.taobao.com'</span>)&nbsp;
input_first&nbsp;=&nbsp;browser.find_element_by_id(<span class="hljs-string">'q'</span>)&nbsp;
input_second&nbsp;=&nbsp;browser.find_element_by_css_selector(<span class="hljs-string">'#q'</span>)&nbsp;
input_third&nbsp;=&nbsp;browser.find_element_by_xpath(<span class="hljs-string">'//*[@id="q"]'</span>)&nbsp;
print(input_first,&nbsp;input_second,&nbsp;input_third)&nbsp;
browser.close()
</code></pre>
<p data-nodeid="108827">这里我们使用 3 种方式获取输入框，分别是根据 id、CSS 选择器和 XPath 获取，它们返回的结果完全一致。运行结果如下：</p>
<pre class="lang-python" data-nodeid="108828"><code data-language="python">&lt;selenium.webdriver.remote.webelement.WebElement&nbsp;(session=<span class="hljs-string">"5e53d9e1c8646e44c14c1c2880d424af"</span>,
&nbsp;element=<span class="hljs-string">"0.5649563096161541-1"</span>)&gt;
&nbsp;
&nbsp;&lt;selenium.webdriver.remote.webelement.WebElement&nbsp;(session
&nbsp;=<span class="hljs-string">"5e53d9e1c8646e44c14c1c2880d424af"</span>,&nbsp;
&nbsp;element=<span class="hljs-string">"0.5649563096161541-1"</span>)&gt;
&nbsp;
&nbsp;&lt;selenium.webdriver.
&nbsp;remote.webelement.WebElement&nbsp;(session=<span class="hljs-string">"5e53d9e1c8646e44c14c1c2880d424af"</span>,&nbsp;
&nbsp;element=<span class="hljs-string">"0.5649563096161541-1"</span>)&gt;
</code></pre>
<p data-nodeid="108829">可以看到，这 3 个节点的类型是一致的，都是 WebElement。</p>
<p data-nodeid="108830">这里列出所有获取单个节点的方法：</p>
<pre class="lang-python" data-nodeid="108831"><code data-language="python">find_element_by_id&nbsp;
find_element_by_name&nbsp;
find_element_by_xpath&nbsp;
find_element_by_link_text&nbsp;
find_element_by_partial_link_text&nbsp;
find_element_by_tag_name&nbsp;
find_element_by_class_name&nbsp;
find_element_by_css_selector
</code></pre>
<p data-nodeid="108832">另外，Selenium 还提供了 find_element 这个通用方法，它需要传入两个参数：查找方式 By 和值。实际上，find_element 就是 find_element_by_id 这种方法的通用函数版本，比如 find_element_by_id(id) 就等价于 find_element(By.ID, id)，二者得到的结果完全一致。我们用代码实现一下：</p>
<pre class="lang-python" data-nodeid="108833"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.common.by&nbsp;<span class="hljs-keyword">import</span>&nbsp;By
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.taobao.com'</span>)&nbsp;
input_first&nbsp;=&nbsp;browser.find_element(By.ID,&nbsp;<span class="hljs-string">'q'</span>)&nbsp;
print(input_first)&nbsp;
browser.close()
</code></pre>
<p data-nodeid="108834">这种查找方式的功能和上面列举的查找函数完全一致，不过参数更加灵活。</p>
<p data-nodeid="108835"><strong data-nodeid="109066">多个节点</strong></p>
<p data-nodeid="108836">如果在网页中只查找一个目标，那么完全可以用 find_element 方法。但如果有多个节点需要查找，再用 find_element 方法，就只能得到第 1 个节点了。如果要查找所有满足条件的节点，需要用 find_elements 这样的方法。<strong data-nodeid="109077">注意，在这个方法的名称中，element 多了一个 s，注意区分。</strong></p>
<p data-nodeid="108837">举个例子，假如你要查找淘宝左侧导航条的所有条目，就可以这样来实现：</p>
<pre class="lang-python" data-nodeid="108838"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.taobao.com'</span>)&nbsp;
lis&nbsp;=&nbsp;browser.find_elements_by_css_selector(<span class="hljs-string">'.service-bd&nbsp;li'</span>)&nbsp;
print(lis)&nbsp;
browser.close()
</code></pre>
<p data-nodeid="108839">运行结果如下：</p>
<pre class="lang-python" data-nodeid="108840"><code data-language="python">[&lt;selenium.webdriver.remote.webelement.WebElement&nbsp;
(session=<span class="hljs-string">"c26290835d4457ebf7d96bfab3740d19"</span>,&nbsp;element=<span class="hljs-string">"0.09221044033125603-1"</span>)&gt;,
&nbsp;
&lt;selenium.webdriver.remote.webelement.WebElement&nbsp;
(session=<span class="hljs-string">"c26290835d4457ebf7d96bfab3740d19"</span>,&nbsp;element=<span class="hljs-string">"0.09221044033125603-2"</span>)&gt;,

&lt;selenium.webdriver.remote.webelement.WebElement&nbsp;
(session=<span class="hljs-string">"c26290835d4457ebf7d96bfab3740d19"</span>,&nbsp;element=<span class="hljs-string">"0.09221044033125603-3"</span>)&gt;...

&lt;selenium.webdriver.remote.webelement.WebElement&nbsp;
(session=<span class="hljs-string">"c26290835d4457ebf7d96bfab3740d19"</span>,&nbsp;element=<span class="hljs-string">"0.09221044033125603-16"</span>)&gt;]
</code></pre>
<p data-nodeid="108841">这里简化了输出结果，中间部分省略。</p>
<p data-nodeid="108842">可以看到，得到的内容变成了列表类型，列表中的每个节点都是 WebElement 类型。</p>
<p data-nodeid="108843">也就是说，如果我们用 find_element 方法，只能获取匹配的第一个节点，结果是 WebElement 类型。如果用 find_elements 方法，则结果是列表类型，列表中的每个节点是 WebElement 类型。</p>
<p data-nodeid="108844">这里列出所有获取多个节点的方法：</p>
<pre class="lang-python" data-nodeid="108845"><code data-language="python">find_elements_by_id&nbsp;
find_elements_by_name&nbsp;
find_elements_by_xpath&nbsp;
find_elements_by_link_text&nbsp;
find_elements_by_partial_link_text&nbsp;
find_elements_by_tag_name&nbsp;
find_elements_by_class_name&nbsp;
find_elements_by_css_selector
</code></pre>
<p data-nodeid="108846">当然，我们也可以直接用 find_elements 方法来选择，这时可以这样写：</p>
<pre class="lang-python" data-nodeid="108847"><code data-language="python">lis&nbsp;=&nbsp;browser.find_elements(By.CSS_SELECTOR,&nbsp;<span class="hljs-string">'.service-bd&nbsp;li'</span>)
</code></pre>
<p data-nodeid="108848">结果是完全一致的。</p>
<h3 data-nodeid="108849">节点交互</h3>
<p data-nodeid="108850">Selenium 可以驱动浏览器来执行一些操作，或者说可以让浏览器模拟执行一些动作。比较常见的用法有：输入文字时用 send_keys 方法，清空文字时用 clear 方法，点击按钮时用 click 方法。示例如下：</p>
<pre class="lang-python" data-nodeid="108851"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
<span class="hljs-keyword">import</span>&nbsp;time&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.taobao.com'</span>)&nbsp;
input&nbsp;=&nbsp;browser.find_element_by_id(<span class="hljs-string">'q'</span>)&nbsp;
input.send_keys(<span class="hljs-string">'iPhone'</span>)&nbsp;
time.sleep(<span class="hljs-number">1</span>)&nbsp;
input.clear()&nbsp;
input.send_keys(<span class="hljs-string">'iPad'</span>)&nbsp;
button&nbsp;=&nbsp;browser.find_element_by_class_name(<span class="hljs-string">'btn-search'</span>)&nbsp;
button.click()
</code></pre>
<p data-nodeid="108852">这里首先驱动浏览器打开淘宝，用 find_element_by_id 方法获取输入框，然后用 send_keys 方法输入 iPhone 文字，等待一秒后用 clear 方法清空输入框，接着再次调用 send_keys 方法输入 iPad 文字，之后再用 find_element_by_class_name 方法获取搜索按钮，最后调用 click 方法完成搜索动作。</p>
<p data-nodeid="108853">通过上面的方法，我们就完成了一些常见节点的动作操作，更多的操作可以参见官方文档的交互动作介绍 ：<a href="http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webelement" data-nodeid="109118">http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webelement</a>。</p>
<h3 data-nodeid="108854">动作链</h3>
<p data-nodeid="108855">在上面的实例中，一些交互动作都是针对某个节点执行的。比如，对于输入框，我们调用它的输入文字和清空文字方法；对于按钮，我们调用它的点击方法。其实，还有另外一些操作，它们没有特定的执行对象，比如鼠标拖拽、键盘按键等，这些动作用另一种方式来执行，那就是动作链。</p>
<p data-nodeid="108856">比如，现在我要实现一个节点的拖拽操作，将某个节点从一处拖拽到另外一处，可以这样实现：</p>
<pre class="lang-python" data-nodeid="108857"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver&nbsp;<span class="hljs-keyword">import</span>&nbsp;ActionChains&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
url&nbsp;=&nbsp;<span class="hljs-string">'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'</span>&nbsp;
browser.get(url)&nbsp;
browser.switch_to.frame(<span class="hljs-string">'iframeResult'</span>)&nbsp;
source&nbsp;=&nbsp;browser.find_element_by_css_selector(<span class="hljs-string">'#draggable'</span>)&nbsp;
target&nbsp;=&nbsp;browser.find_element_by_css_selector(<span class="hljs-string">'#droppable'</span>)&nbsp;
actions&nbsp;=&nbsp;ActionChains(browser)&nbsp;
actions.drag_and_drop(source,&nbsp;target)&nbsp;
actions.perform()
</code></pre>
<p data-nodeid="108858">首先，打开网页中的一个拖拽实例，依次选中要拖拽的节点和拖拽到的目标节点，接着声明 ActionChains 对象并将其赋值为 actions 变量，然后通过调用 actions 变量的 drag_and_drop 方法，再调用 perform 方法执行动作，此时就完成了拖拽操作，如图所示：</p>
<p data-nodeid="108859"><img src="https://s0.lgstatic.com/i/image3/M01/7D/12/Cgq2xl59oBaAebZXAACbaBgWl4k530.png" alt="" data-nodeid="109129"></p>
<p data-nodeid="108860">拖拽前页面<br>
<img src="https://s0.lgstatic.com/i/image3/M01/03/FC/Ciqah159oBeAZICwAACKn0bkfog611.png" alt="" data-nodeid="109133"></p>
<p data-nodeid="108861">拖拽后页面</p>
<p data-nodeid="108862">以上两图分别为在拖拽前和拖拽后的结果。</p>
<p data-nodeid="108863">更多的动作链操作可以参考官方文档的动作链介绍：<a href="http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.common.action_chains" data-nodeid="109141">http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.common.action_chains</a>。</p>
<h3 data-nodeid="108864">执行 JavaScript</h3>
<p data-nodeid="108865">Selenium API 并没有提供实现某些操作的方法，比如，下拉进度条。但它可以直接模拟运行 JavaScript，此时使用 execute_script 方法即可实现，代码如下：</p>
<pre class="lang-js" data-nodeid="108866"><code data-language="js"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.zhihu.com/explore'</span>)&nbsp;
browser.execute_script(<span class="hljs-string">'window.scrollTo(0,&nbsp;document.body.scrollHeight)'</span>)&nbsp;
browser.execute_script(<span class="hljs-string">'alert("To&nbsp;Bottom")'</span>)
</code></pre>
<p data-nodeid="108867">这里利用 execute_script 方法将进度条下拉到最底部，然后弹出 alert 提示框。</p>
<p data-nodeid="108868">有了这个方法，基本上 API 没有提供的所有功能都可以用执行 JavaScript 的方式来实现了。</p>
<h3 data-nodeid="108869">获取节点信息</h3>
<p data-nodeid="108870">前面说过，通过 page_source 属性可以获取网页的源代码，接着就可以使用解析库（如正则表达式、Beautiful Soup、pyquery 等）来提取信息了。</p>
<p data-nodeid="108871">不过，既然 Selenium 已经提供了选择节点的方法，并且返回的是 WebElement 类型，那么它也有相关的方法和属性来直接提取节点信息，如属性、文本等。这样的话，我们就可以不用通过解析源代码来提取信息了，非常方便。</p>
<p data-nodeid="108872">接下来，我们就来看看可以通过怎样的方式来获取节点信息吧。</p>
<p data-nodeid="108873"><strong data-nodeid="109160">获取属性</strong></p>
<p data-nodeid="108874">我们可以使用 get_attribute 方法来获取节点的属性，但是前提是得先选中这个节点，示例如下：</p>
<pre class="lang-python" data-nodeid="111565"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
url&nbsp;=&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/'</span>&nbsp;
browser.get(url)&nbsp;
logo&nbsp;=&nbsp;browser.find_element_by_class_name(<span class="hljs-string">'logo-image'</span>)
print(logo)&nbsp;
print(logo.get_attribute(<span class="hljs-string">'src'</span>))
</code></pre>

<p data-nodeid="108876">运行之后，程序便会驱动浏览器打开该页面，然后获取 class 为 logo-image 的节点，最后打印出它的 src 属性。</p>
<p data-nodeid="108877">控制台的输出结果如下：</p>
<pre class="lang-python" data-nodeid="113779"><code data-language="python">&lt;selenium.webdriver.remote.webelement.WebElement&nbsp;
(session=<span class="hljs-string">"7f4745d35a104759239b53f68a6f27d0"</span>,&nbsp;
element=<span class="hljs-string">"cd7c72b4-4920-47ed-91c5-ea06601dc509"</span>)&gt;&nbsp;
https://dynamic2.scrape.center/img/logo.a508a8f0.png
</code></pre>


<p data-nodeid="108879">通过 get_attribute 方法，我们只需要传入想要获取的属性名，就可以得到它的值了。</p>
<p data-nodeid="108880"><strong data-nodeid="109172">获取文本值</strong></p>
<p data-nodeid="108881">每个 WebElement 节点都有 text 属性，直接调用这个属性就可以得到节点内部的文本信息，这相当于 pyquery 的 text 方法，示例如下：</p>
<pre class="lang-python" data-nodeid="114886"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()
url&nbsp;=&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/'</span>&nbsp;
browser.get(url)
input&nbsp;=&nbsp;browser.find_element_by_class_name(<span class="hljs-string">'logo-title'</span>)&nbsp;
print(input.text)
</code></pre>

<p data-nodeid="108883">这里依然先打开页面，然后获取 class 为 logo-title 这个节点，再将其文本值打印出来。</p>
<p data-nodeid="108884">控制台的输出结果如下：</p>
<pre data-nodeid="108885"><code>Scrape
</code></pre>
<h3 data-nodeid="108886">获取 ID、位置、标签名、大小</h3>
<p data-nodeid="108887">另外，WebElement 节点还有一些其他属性，比如 id 属性可以获取节点 id，location 属性可以获取该节点在页面中的相对位置，tag_name 属性可以获取标签名称，size 属性可以获取节点的大小，也就是宽高，这些属性有时候还是很有用的。示例如下：</p>
<pre class="lang-python" data-nodeid="115993"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
url&nbsp;=&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/'</span>&nbsp;
browser.get(url)&nbsp;
input&nbsp;=&nbsp;browser.find_element_by_class_name(<span class="hljs-string">'logo-title'</span>)&nbsp;
print(input.id)&nbsp;
print(input.location)&nbsp;
print(input.tag_name)&nbsp;
print(input.size)
</code></pre>

<p data-nodeid="108889">这里首先获得 class 为 logo-title 这个节点，然后调用其 id、location、tag_name、size 属性来获取对应的属性值。</p>
<h3 data-nodeid="108890">切换 Frame</h3>
<p data-nodeid="108891">我们知道网页中有一种节点叫作 iframe，也就是子 Frame，相当于页面的子页面，它的结构和外部网页的结构完全一致。Selenium 打开页面后，默认是在父级 Frame 里面操作，而此时如果页面中还有子 Frame，Selenium 是不能获取到子 Frame 里面的节点的。这时就需要使用 switch_to.frame 方法来切换 Frame。示例如下：</p>
<pre class="lang-python" data-nodeid="108892"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;time&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.common.exceptions&nbsp;<span class="hljs-keyword">import</span>&nbsp;NoSuchElementException&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()
url&nbsp;=&nbsp;<span class="hljs-string">'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'</span>&nbsp;
browser.get(url)&nbsp;
browser.switch_to.frame(<span class="hljs-string">'iframeResult'</span>)
<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;logo&nbsp;=&nbsp;browser.find_element_by_class_name(<span class="hljs-string">'logo'</span>)&nbsp;
<span class="hljs-keyword">except</span>&nbsp;NoSuchElementException:
&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'NO&nbsp;LOGO'</span>)&nbsp;
browser.switch_to.parent_frame()&nbsp;
logo&nbsp;=&nbsp;browser.find_element_by_class_name(<span class="hljs-string">'logo'</span>)
print(logo)&nbsp;
print(logo.text)
</code></pre>
<p data-nodeid="108893">控制台输出：</p>
<pre class="lang-pytohn" data-nodeid="108894"><code data-language="pytohn">NO&nbsp;LOGO&nbsp;
&lt;selenium.webdriver.remote.webelement.WebElement
(session="4bb8ac03ced4ecbdefef03ffdc0e4ccd",&nbsp;
element="0.13792611320464965-2")&gt;&nbsp;
RUNOOB.COM
</code></pre>
<p data-nodeid="108895">这里还是以前面演示动作链操作的网页为例，首先通过 switch_to.frame 方法切换到子 Frame 里面，然后尝试获取子 Frame 里的 logo 节点（这是不能找到的），如果找不到的话，就会抛出 NoSuchElementException 异常，异常被捕捉之后，就会输出 NO LOGO。接下来，我们需要重新切换回父级 Frame，然后再次重新获取节点，发现此时可以成功获取了。</p>
<p data-nodeid="108896">所以，当页面中包含子 Frame 时，如果想获取子 Frame 中的节点，需要先调用 switch_to.frame 方法切换到对应的 Frame，然后再进行操作。</p>
<h3 data-nodeid="108897">延时等待</h3>
<p data-nodeid="108898">在 Selenium 中，get 方法会在网页框架加载结束后结束执行，此时如果获取 page_source，可能并不是浏览器完全加载完成的页面，如果某些页面有额外的 Ajax 请求，我们在网页源代码中也不一定能成功获取到。所以，这里需要延时等待一定时间，确保节点已经加载出来。</p>
<p data-nodeid="108899">这里等待的方式有两种：一种是隐式等待，一种是显式等待。</p>
<p data-nodeid="108900"><strong data-nodeid="109202">隐式等待</strong></p>
<p data-nodeid="108901">当使用隐式等待执行测试的时候，如果 Selenium 没有在 DOM 中找到节点，将继续等待，超出设定时间后，则抛出找不到节点的异常。换句话说，隐式等待可以在我们查找节点而节点并没有立即出现的时候，等待一段时间再查找 DOM，默认的时间是 0。示例如下：</p>
<pre class="lang-python" data-nodeid="118207"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.implicitly_wait(<span class="hljs-number">10</span>)&nbsp;
browser.get(<span class="hljs-string">'https://dynamic2.scrape.center/'</span>)&nbsp;
input&nbsp;=&nbsp;browser.find_element_by_class_name(<span class="hljs-string">'logo-image'</span>)&nbsp;
print(input)
</code></pre>


<p data-nodeid="108903">在这里我们用 implicitly_wait 方法实现了隐式等待。</p>
<p data-nodeid="108904"><strong data-nodeid="109210">显式等待</strong></p>
<p data-nodeid="108905">隐式等待的效果其实并没有那么好，因为我们只规定了一个固定时间，而页面的加载时间会受到网络条件的影响。</p>
<p data-nodeid="108906">这里还有一种更合适的显式等待方法，它指定要查找的节点，然后指定一个最长等待时间。如果在规定时间内加载出来了这个节点，就返回查找的节点；如果到了规定时间依然没有加载出该节点，则抛出超时异常。示例如下：</p>
<pre class="lang-python" data-nodeid="108907"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.common.by&nbsp;<span class="hljs-keyword">import</span>&nbsp;By&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.support.ui&nbsp;<span class="hljs-keyword">import</span>&nbsp;WebDriverWait&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.support&nbsp;<span class="hljs-keyword">import</span>&nbsp;expected_conditions&nbsp;<span class="hljs-keyword">as</span>&nbsp;EC&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.taobao.com/'</span>)&nbsp;
wait&nbsp;=&nbsp;WebDriverWait(browser,&nbsp;<span class="hljs-number">10</span>)&nbsp;
input&nbsp;=&nbsp;wait.until(EC.presence_of_element_located((By.ID,&nbsp;<span class="hljs-string">'q'</span>)))&nbsp;
button&nbsp;=&nbsp;&nbsp;wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR,&nbsp;<span class="hljs-string">'.btn-search'</span>)))&nbsp;
print(input,&nbsp;button)
</code></pre>
<p data-nodeid="108908">这里首先引入 WebDriverWait 这个对象，指定最长等待时间，然后调用它的 until() 方法，传入要等待条件 expected_conditions。比如，这里传入了 presence_of_element_located 这个条件，代表节点出现，其参数是节点的定位元组，也就是 ID 为 q 的节点搜索框。</p>
<p data-nodeid="108909">这样做的效果就是，在 10 秒内如果 ID 为 q 的节点（即搜索框）成功加载出来，就返回该节点；如果超过 10 秒还没有加载出来，就抛出异常。</p>
<p data-nodeid="108910">对于按钮，我们可以更改一下等待条件，比如改为 element_to_be_clickable，也就是可点击，所以查找按钮时先查找 CSS 选择器为.btn-search 的按钮，如果 10 秒内它是可点击的，也就代表它成功加载出来了，就会返回这个按钮节点；如果超过 10 秒还不可点击，也就是没有加载出来，就抛出异常。</p>
<p data-nodeid="108911">现在我们运行代码，它在网速较佳的情况下是可以成功加载出来的。</p>
<p data-nodeid="108912">控制台的输出如下：</p>
<pre class="lang-python" data-nodeid="108913"><code data-language="python">&lt;selenium.webdriver.remote.webelement.WebElement&nbsp;
(session=<span class="hljs-string">"07dd2fbc2d5b1ce40e82b9754aba8fa8"</span>,&nbsp;
element=<span class="hljs-string">"0.5642646294074107-1"</span>)&gt;
&lt;selenium.webdriver.remote.webelement.WebElement&nbsp;
(session=<span class="hljs-string">"07dd2fbc2d5b1ce40e82b9754aba8fa8"</span>,&nbsp;
element=<span class="hljs-string">"0.5642646294074107-2"</span>)&gt;
</code></pre>
<p data-nodeid="108914">可以看到，控制台成功输出了两个节点，它们都是 WebElement 类型。</p>
<p data-nodeid="108915">如果网络有问题，10 秒内没有成功加载，那就抛出 TimeoutException 异常，此时控制台的输出如下：</p>
<pre class="lang-python" data-nodeid="108916"><code data-language="python">TimeoutException&nbsp;Traceback&nbsp;(most&nbsp;recent&nbsp;call&nbsp;last)&nbsp;
&lt;ipython-input-4-f3d73973b223&gt;&nbsp;in&nbsp;&lt;module&gt;()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7&nbsp;browser.get('https://www.taobao.com/')
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8&nbsp;wait&nbsp;=&nbsp;WebDriverWait(browser,&nbsp;10)&nbsp;
----&gt;&nbsp;9&nbsp;input&nbsp;=&nbsp;wait.until(EC.presence_of_element_located((By.ID,&nbsp;'q')))
</code></pre>
<p data-nodeid="108917">关于等待条件，其实还有很多，比如判断标题内容，判断某个节点内是否出现了某文字等。下表我列出了所有的等待条件。</p>
<p data-nodeid="108918"><img src="https://s0.lgstatic.com/i/image3/M01/04/3A/Ciqah1596FyAIAjtAAECe0Jujuw745.png" alt="" data-nodeid="109236"></p>
<p data-nodeid="108919"><img src="https://s0.lgstatic.com/i/image3/M01/04/3A/Ciqah1596R2Af973AAEiFfxC3E4161.png" alt="" data-nodeid="109238"></p>
<p data-nodeid="108920">更多详细的等待条件的参数及用法介绍可以参考官方文档：<a href="http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.support.expected_conditions" data-nodeid="109244">http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.support.expected_conditions</a>。</p>
<h3 data-nodeid="108921">前进后退</h3>
<p data-nodeid="108922">平常我们使用浏览器时都有前进和后退功能，Selenium 也可以完成这个操作，它使用 back 方法后退，使用 forward 方法前进。示例如下：</p>
<pre class="lang-python" data-nodeid="108923"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;time&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.baidu.com/'</span>)&nbsp;
browser.get(<span class="hljs-string">'https://www.taobao.com/'</span>)&nbsp;
browser.get(<span class="hljs-string">'https://www.python.org/'</span>)&nbsp;
browser.back()&nbsp;
time.sleep(<span class="hljs-number">1</span>)&nbsp;
browser.forward()&nbsp;
browser.close()
</code></pre>
<p data-nodeid="108924">这里我们连续访问 3 个页面，然后调用 back &nbsp;方法回到第 2 个页面，接下来再调用 forward 方法又可以前进到第 3 个页面。</p>
<h3 data-nodeid="108925">Cookies</h3>
<p data-nodeid="108926">使用 Selenium，还可以方便地对 Cookies 进行操作，例如获取、添加、删除 Cookies 等。示例如下：</p>
<pre class="lang-python" data-nodeid="108927"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.zhihu.com/explore'</span>)&nbsp;
print(browser.get_cookies())&nbsp;
browser.add_cookie({<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'name'</span>,&nbsp;<span class="hljs-string">'domain'</span>:&nbsp;<span class="hljs-string">'www.zhihu.com'</span>,&nbsp;<span class="hljs-string">'value'</span>:&nbsp;<span class="hljs-string">'germey'</span>})&nbsp;
print(browser.get_cookies())&nbsp;
browser.delete_all_cookies()&nbsp;
print(browser.get_cookies())
</code></pre>
<p data-nodeid="108928">首先，我们访问知乎，加载完成后，浏览器实际上已经生成 Cookies 了。接着，调用 get_cookies 方法获取所有的 Cookies。然后，我们再添加一个 Cookie，这里传入一个字典，有 name、domain 和 value 等内容。接下来，再次获取所有的 Cookies，可以发现，结果会多出这一项新加的 Cookie。最后，调用 delete_all_cookies 方法删除所有的 Cookies。再重新获取，发现结果就为空了。</p>
<p data-nodeid="108929">控制台的输出如下：</p>
<pre class="lang-python" data-nodeid="108930"><code data-language="python">[{<span class="hljs-string">'secure'</span>:&nbsp;<span class="hljs-literal">False</span>,&nbsp;
<span class="hljs-string">'value'</span>:&nbsp;<span class="hljs-string">'"NGM0ZTM5NDAwMWEyNDQwNDk5ODlkZWY3OTkxY2I0NDY=|1491604091|236e34290a6f407bfbb517888849ea509ac366d0"'</span>,&nbsp;
<span class="hljs-string">'domain'</span>:&nbsp;<span class="hljs-string">'.zhihu.com'</span>,
<span class="hljs-string">'path'</span>:&nbsp;<span class="hljs-string">'/'</span>,&nbsp;
<span class="hljs-string">'httpOnly'</span>:&nbsp;<span class="hljs-literal">False</span>,&nbsp;
<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'l_cap_id'</span>,&nbsp;
<span class="hljs-string">'expiry'</span>:&nbsp;<span class="hljs-number">1494196091.403418</span>},...]&nbsp;
[{<span class="hljs-string">'secure'</span>:&nbsp;<span class="hljs-literal">False</span>,&nbsp;
<span class="hljs-string">'value'</span>:&nbsp;<span class="hljs-string">'germey'</span>,&nbsp;
<span class="hljs-string">'domain'</span>:&nbsp;<span class="hljs-string">'.www.zhihu.com'</span>,&nbsp;
<span class="hljs-string">'path'</span>:&nbsp;<span class="hljs-string">'/'</span>,&nbsp;
<span class="hljs-string">'httpOnly'</span>:&nbsp;<span class="hljs-literal">False</span>,&nbsp;
<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'name'</span>},&nbsp;
{<span class="hljs-string">'secure'</span>:&nbsp;<span class="hljs-literal">False</span>,&nbsp;
<span class="hljs-string">'value'</span>:&nbsp;<span class="hljs-string">'"NGM0ZTM5NDAwMWEyNDQwNDk5ODlkZWY3OTkxY2I0NDY=|1491604091|236e34290a6f407bfbb517888849ea509ac366d0"'</span>,&nbsp;
<span class="hljs-string">'domain'</span>:&nbsp;<span class="hljs-string">'.zhihu.com'</span>,&nbsp;
<span class="hljs-string">'path'</span>:<span class="hljs-string">'/'</span>,&nbsp;
<span class="hljs-string">'httpOnly'</span>:&nbsp;<span class="hljs-literal">False</span>,&nbsp;
<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'l_cap_id'</span>,&nbsp;
<span class="hljs-string">'expiry'</span>:&nbsp;<span class="hljs-number">1494196091.403418</span>},&nbsp;...]&nbsp;
[]
</code></pre>
<p data-nodeid="108931">通过以上方法来操作 Cookies 还是非常方便的。</p>
<h3 data-nodeid="108932">选项卡管理</h3>
<p data-nodeid="108933">在访问网页的时候，我们通常会开启多个选项卡。在 Selenium 中，我们也可以对选项卡进行操作。示例如下：</p>
<pre class="lang-python" data-nodeid="108934"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;time&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.baidu.com'</span>)&nbsp;
browser.execute_script(<span class="hljs-string">'window.open()'</span>)&nbsp;
print(browser.window_handles)&nbsp;
browser.switch_to.window(browser.window_handles[<span class="hljs-number">1</span>])
browser.get(<span class="hljs-string">'https://www.taobao.com'</span>)&nbsp;
time.sleep(<span class="hljs-number">1</span>)&nbsp;
browser.switch_to.window(browser.window_handles[<span class="hljs-number">0</span>])&nbsp;
browser.get(<span class="hljs-string">'https://python.org'</span>
</code></pre>
<p data-nodeid="108935">控制台输出如下：</p>
<pre class="lang-python" data-nodeid="108936"><code data-language="python">[<span class="hljs-string">'CDwindow-4f58e3a7-7167-4587-bedf-9cd8c867f435'</span>,&nbsp;<span class="hljs-string">'CDwindow-6e05f076-6d77-453a-a36c-32baacc447df'</span>]
</code></pre>
<p data-nodeid="108937">首先访问百度，然后调用 execute_script 方法，这里我们传入 window.open 这个 JavaScript 语句新开启一个选项卡，然后切换到该选项卡，调用 window_handles 属性获取当前开启的所有选项卡，后面的参数代表返回选项卡的代号列表。要想切换选项卡，只需要调用 switch_to.window 方法即可，其中的参数是选项卡的代号。这里我们将第 2 个选项卡代号传入，即跳转到第 2 个选项卡，接下来在第 2 个选项卡下打开一个新页面，如果你想要切换回第 2 个选项卡，只需要重新调用 switch_to.window 方法，再执行其他操作即可。</p>
<h3 data-nodeid="108938">异常处理</h3>
<p data-nodeid="108939">在使用 Selenium 的过程中，难免会遇到一些异常，例如超时、节点未找到等错误，一旦出现此类错误，程序便不会继续运行了。这里我们可以使用 try except 语句来捕获各种异常。</p>
<p data-nodeid="108940">首先，演示一下节点未找到的异常，示例如下：</p>
<pre class="lang-python" data-nodeid="108941"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()&nbsp;
browser.get(<span class="hljs-string">'https://www.baidu.com'</span>)&nbsp;
browser.find_element_by_id(<span class="hljs-string">'hello'</span>)
</code></pre>
<p data-nodeid="108942">这里我们首先打开百度页面，然后尝试选择一个并不存在的节点，此时就会遇到异常。</p>
<p data-nodeid="108943">运行之后控制台的输出如下：</p>
<pre class="lang-python" data-nodeid="108944"><code data-language="python">NoSuchElementException&nbsp;Traceback&nbsp;(most&nbsp;recent&nbsp;call&nbsp;last)&nbsp;
&lt;ipython-input-23-978945848a1b&gt;&nbsp;in&nbsp;&lt;module&gt;()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3&nbsp;browser&nbsp;=&nbsp;webdriver.Chrome()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4&nbsp;browser.get&nbsp;('https://www.baidu.com')
----&gt;&nbsp;5&nbsp;browser.find_element_by_id('hello')
</code></pre>
<p data-nodeid="108945">可以看到，这里抛出了 NoSuchElementException 异常，通常代表节点未找到。为了防止程序遇到异常而中断，我们需要捕获这些异常，示例如下：</p>
<pre class="lang-python" data-nodeid="108946"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver&nbsp;
<span class="hljs-keyword">from</span>&nbsp;selenium.common.exceptions&nbsp;<span class="hljs-keyword">import</span>&nbsp;TimeoutException,&nbsp;
NoSuchElementException&nbsp;
browser&nbsp;=&nbsp;webdriver.Chrome()
<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;browser.get(<span class="hljs-string">'https://www.baidu.com'</span>)&nbsp;
<span class="hljs-keyword">except</span>&nbsp;TimeoutException:
&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'Time&nbsp;Out'</span>)&nbsp;
<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;browser.find_element_by_id(<span class="hljs-string">'hello'</span>)&nbsp;
<span class="hljs-keyword">except</span>&nbsp;NoSuchElementException:
&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'No&nbsp;Element'</span>)&nbsp;
<span class="hljs-keyword">finally</span>:
&nbsp;&nbsp;&nbsp;&nbsp;browser.close()
</code></pre>
<p data-nodeid="108947">这里我们使用 try except 来捕获各类异常。比如，我们用 find_element_by_id 查找节点的方法捕获 NoSuchElementException 异常，这样一旦出现这样的错误，就进行异常处理，程序也不会中断了。</p>
<p data-nodeid="108948">控制台的输出如下：</p>
<pre data-nodeid="108949"><code>No&nbsp;Element
</code></pre>
<p data-nodeid="108950">关于更多的异常类，可以参考官方文档：：<a href="http://selenium-python.readthedocs.io/api.html#module-selenium.common.exceptions" data-nodeid="109289">http://selenium-python.readthedocs.io/api.html#module-selenium.common.exceptions</a>。</p>
<h3 data-nodeid="108951">反屏蔽</h3>
<p data-nodeid="108952">现在很多网站都加上了对 Selenium 的检测，来防止一些爬虫的恶意爬取。即如果检测到有人在使用 Selenium 打开浏览器，那就直接屏蔽。</p>
<p data-nodeid="108953">其大多数情况下，检测基本原理是检测当前浏览器窗口下的 window.navigator 对象是否包含 webdriver 这个属性。因为在正常使用浏览器的情况下，这个属性是 undefined，然而一旦我们使用了 Selenium，Selenium 会给 window.navigator 设置 webdriver 属性。很多网站就通过 JavaScript 判断如果 webdriver 属性存在，那就直接屏蔽。</p>
<p data-nodeid="120426" class="">这边有一个典型的案例网站：<a href="https://antispider1.scrape.center/" data-nodeid="120430">https://antispider1.scrape.center/</a>，这个网站就是使用了上述原理实现了 WebDriver 的检测，如果使用 Selenium 直接爬取的话，那就会返回如下页面：</p>


<p data-nodeid="108955"><img src="https://s0.lgstatic.com/i/image3/M01/7D/12/Cgq2xl59oBeATES6AABGMW_83Oc577.png" alt="" data-nodeid="109300"></p>
<p data-nodeid="108956">这时候我们可能想到直接使用 JavaScript 直接把这个 webdriver 属性置空，比如通过调用 execute_script 方法来执行如下代码：</p>
<pre class="lang-python" data-nodeid="108957"><code data-language="python">Object.defineProperty(navigator,&nbsp;"webdriver",&nbsp;{get:&nbsp;()&nbsp;=&gt;&nbsp;undefined})
</code></pre>
<p data-nodeid="108958">这行 JavaScript 的确是可以把 webdriver 属性置空，但是 execute_script 调用这行 JavaScript 语句实际上是在页面加载完毕之后才执行的，执行太晚了，网站早在最初页面渲染之前就已经对 webdriver 属性进行了检测，所以用上述方法并不能达到效果。</p>
<p data-nodeid="108959">在 Selenium 中，我们可以使用 CDP（即 Chrome Devtools-Protocol，Chrome 开发工具协议）来解决这个问题，通过 CDP 我们可以实现在每个页面刚加载的时候执行 JavaScript 代码，执行的 CDP 方法叫作 Page.addScriptToEvaluateOnNewDocument，然后传入上文的 JavaScript 代码即可，这样我们就可以在每次页面加载之前将 webdriver 属性置空了。另外我们还可以加入几个选项来隐藏 WebDriver 提示条和自动化扩展信息，代码实现如下：</p>
<pre class="lang-python te-preview-highlight" data-nodeid="121538"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver&nbsp;<span class="hljs-keyword">import</span>&nbsp;ChromeOptions

option&nbsp;=&nbsp;ChromeOptions()
option.add_experimental_option(<span class="hljs-string">'excludeSwitches'</span>,&nbsp;[<span class="hljs-string">'enable-automation'</span>])
option.add_experimental_option(<span class="hljs-string">'useAutomationExtension'</span>,&nbsp;<span class="hljs-literal">False</span>)
browser&nbsp;=&nbsp;webdriver.Chrome(options=option)
browser.execute_cdp_cmd(<span class="hljs-string">'Page.addScriptToEvaluateOnNewDocument'</span>,&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-string">'source'</span>:&nbsp;<span class="hljs-string">'Object.defineProperty(navigator,&nbsp;"webdriver",&nbsp;{get:&nbsp;()&nbsp;=&gt;&nbsp;undefined})'</span>
})
browser.get(<span class="hljs-string">'https://antispider1.scrape.center/'</span>)
</code></pre>

<p data-nodeid="108961">这样整个页面就能被加载出来了：</p>
<p data-nodeid="108962"><img src="https://s0.lgstatic.com/i/image3/M01/03/FC/Ciqah159oBeAHd49AAMjMlBnsHE279.png" alt="" data-nodeid="109310"></p>
<p data-nodeid="108963">对于大多数的情况，以上的方法均可以实现 Selenium 反屏蔽。但对于一些特殊的网站，如果其有更多的 WebDriver 特征检测，可能需要具体排查。</p>
<h3 data-nodeid="108964">无头模式</h3>
<p data-nodeid="108965">上面的案例在运行的时候，我们可以观察到其总会弹出一个浏览器窗口，虽然有助于观察页面爬取状况，但在有些时候窗口弹来弹去也会形成一些干扰。</p>
<p data-nodeid="108966">Chrome 浏览器从 60 版本已经支持了无头模式，即 Headless。无头模式在运行的时候不会再弹出浏览器窗口，减少了干扰，而且它减少了一些资源的加载，如图片等资源，所以也在一定程度上节省了资源加载时间和网络带宽。</p>
<p data-nodeid="108967">我们可以借助于 ChromeOptions 来开启 Chrome Headless 模式，代码实现如下：</p>
<pre class="lang-python" data-nodeid="108968"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver&nbsp;<span class="hljs-keyword">import</span>&nbsp;ChromeOptions

option&nbsp;=&nbsp;ChromeOptions()
option.add_argument(<span class="hljs-string">'--headless'</span>)
browser&nbsp;=&nbsp;webdriver.Chrome(options=option)
browser.set_window_size(<span class="hljs-number">1366</span>,&nbsp;<span class="hljs-number">768</span>)
browser.get(<span class="hljs-string">'https://www.baidu.com'</span>)
browser.get_screenshot_as_file(<span class="hljs-string">'preview.png'</span>)
</code></pre>
<p data-nodeid="108969">这里我们通过 ChromeOptions 的 add_argument 方法添加了一个参数 --headless，开启了无头模式。在无头模式下，我们最好需要设置下窗口的大小，接着打开页面，最后我们调用 get_screenshot_as_file 方法输出了页面的截图。</p>
<p data-nodeid="108970">运行代码之后，我们发现 Chrome 窗口就不会再弹出来了，代码依然正常运行，最后输出了页面截图如图所示。</p>
<p data-nodeid="108971"><img src="https://s0.lgstatic.com/i/image3/M01/7D/12/Cgq2xl59oBeAdYtPAACc0m2Jx3Y415.png" alt="" data-nodeid="109327"></p>
<p data-nodeid="108972">这样我们就在无头模式下完成了页面的抓取和截图操作。</p>
<p data-nodeid="108973">现在，我们基本对 Selenium 的常规用法有了大体的了解。使用 Selenium，处理 JavaScript 渲染的页面不再是难事。</p>
<p data-nodeid="108974">本课时课的内容就到这里，&lt;span style="color:#333333"&gt; 面我们会用一个实例来演示 Selenium 爬取网站的流程，记得按时来听课哟。</p>
<p data-nodeid="108975" class="">本节代码：<a href="https://github.com/Python3WebSpider/SeleniumTest" data-nodeid="109340">https://github.com/Python3WebSpider/SeleniumTest</a></p>

---

### 精选评论

##### **3785：
> 这里的selenium爬取某宝失效了，必要要登录，某宝做了反爬

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 关于反爬的内容后面也会讲到哦，加油

##### *强：
> 崔老师 上周用你的规避方法 淘宝无法识别 这周就被检测出来了 可否 再发下更好的规避方法<div><br></div><div>？谢谢</div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个淘宝更新比较猛，可能又校验了其他的浏览器指纹，这个具体再搜搜看吧。比如 selenium爬虫被检测到 该如何破？ - Angel的回答 - 知乎
https://www.zhihu.com/question/50738719/answer/545145218 这里面的一些字段，或者替换下 cdc https://stackoverflow.com/questions/33225947/can-a-website-detect-when-you-are-using-selenium-with-chromedriver

##### bing wu：
> 老师以后多增加点实战项目如何，想配合您那本书结合这个教程进一步进阶一下🙏🙏

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 课程讲解是循序渐进的，后面的课时会涉及大量的爬虫实操案例

##### *秦：
> 这节课内容好多，需要好好实践，消化！

##### **阳：
> 现在想爬取大量的赌博网站（需要登录），但是网站格式不太一样，这种做通用爬虫有什么思路吗<div><br></div>

##### rigel：
> 想请问下老师，无头模式下为什么最好需要设置下窗口的大小？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 无头模式的默认窗口大小比较小，有时候并不是我们平常浏览器的大小，所以建议设置窗口大小。

##### 李：
> 老师，反屏蔽那里，用您提供代码，提示unknown command错误

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个检查下是哪个 command，看看是不是对应的环境变量没配置上。

##### *念：
> 关于Selenium反屏蔽，基本方法是防止网站监测到Selenium WebDriver的特征，那可不可以通过编译安装Selenium，只保留模拟浏览器基本功能，而把其他特征都消除掉呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是可以的，但工程量比较大，可以试试。

##### *强：
> 用Selenium爬到第八页报输入验证码，手动输入无效怎么解决？<div><br></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个可能也是检测了 Selenium 指纹或者你的行为，被反爬了，试着隐藏一些信息看看。

##### **耿爱生活：
> 课件

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 关注拉勾教育公众号 咨询小助手获取课件

