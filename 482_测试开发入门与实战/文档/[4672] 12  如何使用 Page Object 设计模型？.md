<p data-nodeid="84674" class="">到上节课为止，你不仅已经具备了测试开发的初级能力，而且能够搭建起融合 API 自动化测试和 UI 自动化测试的框架。</p>
<p data-nodeid="84675">今天我将带你一起，使用 PageObject 模型，优化我们的框架代码，使我们的框架结构更加清晰，代码更加模块化，以便在迁移项目重用框架时成本更低。</p>
<p data-nodeid="84676">下方是课程内容结构图，可供你学习参考：<br>
<img src="https://s0.lgstatic.com/i/image/M00/60/81/Ciqc1F-NcGWAZc7WAAUL4728Z-8331.png" alt="Lark20201019-184959.png" data-nodeid="84800"></p>
<h3 data-nodeid="84677">什么是 PageObject 设计模型？</h3>
<p data-nodeid="84678">PageObject 设计模型是在自动化测试过程中普遍采用的一种设计模式。它通过对页面对象（实际的 UI 页面，或者是逻辑上的页面）进行抽象，使得你的代码能在页面元素发生改变时，<strong data-nodeid="84807">尽量少地更改，以最大程度地支持代码重用和避免代码冗余</strong>。</p>
<h3 data-nodeid="84679">PageObject 设计模型的特征</h3>
<p data-nodeid="84680">目前，并没有一种统一的格式（format）来告诉你如何设计 PageObject，只要你设计的代码将页面元素和测试代码分离，你都可以说你使用了 PageObject。</p>
<p data-nodeid="84681">一般情况下，实现了 PageObject 的代码往往具备如下特征：</p>
<p data-nodeid="84682"><strong data-nodeid="84814">1.页面封装成 Page 类，页面元素为 Page 类的成员元素，页面功能放在 Page 类方法里。</strong></p>
<p data-nodeid="84683">将一个页面（或者待测试对象）封装成一个类（Class），把它称作 Page 类。Page 类里包括了这个页面（或者待测试对象）上的所有的元素，以及针对页面元素的操作方法（单步操作或者多步操作，一般会定义类方法）。注意：这个Page 类里仅仅包括当前页面，一般不包括针对其他页面的操作。</p>
<p data-nodeid="84684"><strong data-nodeid="84819">2.针对这个 Page 类定义一个测试类，在测试类调用 Page 类的各个类方法完成测试。</strong></p>
<p data-nodeid="84685">也就是测试代码和被测试页面的页面代码解耦，当页面本身发生变化，例如元素定位发生改变、页面布局改变后，仅需要更改相对应的 Page 类的代码，而无须更改测试类的代码。</p>
<p data-nodeid="84686">PageObject 模式减少了代码冗余，可以使业务流程变得清晰易读，降低了测试代码维护成本。</p>
<h3 data-nodeid="84687">PageObject 的实现</h3>
<p data-nodeid="84688">根据上述特点，我们来看下一个 PageObject 的经典设计：</p>
<p data-nodeid="84689"><img src="https://s0.lgstatic.com/i/image/M00/60/8D/CgqCHl-NcHKALdY4AAFp3jzWIEU818.png" alt="Lark20201019-185002.png" data-nodeid="84826"></p>
<p data-nodeid="84690">可以看到，在<strong data-nodeid="84856">测试类</strong>里，我们会定义许多<strong data-nodeid="84857">测试方法</strong>，这些测试方法里，会含有对<strong data-nodeid="84858">页面对象实例</strong>的调用；而<strong data-nodeid="84859">页面对象实例</strong>，是通过<strong data-nodeid="84860">页面对象类</strong>进行初始化操作生成的；对于许多<strong data-nodeid="84861">页面对象类</strong>都存在的通用操作，我们会提取到<strong data-nodeid="84862">页面对象基类</strong>里。</p>
<p data-nodeid="84691">通过这种方法，我们就实现了：</p>
<ul data-nodeid="84692">
<li data-nodeid="84693">
<p data-nodeid="84694">一个页面元素在整个项目中，仅存在一处定义，其他都是调用；</p>
</li>
<li data-nodeid="84695">
<p data-nodeid="84696">Page 类通用的操作进一步提取到 BasePage 类，减少了代码冗余。</p>
</li>
</ul>
<h3 data-nodeid="84697">PageObject 的 Python 库</h3>
<p data-nodeid="84698">在 Python 里，有专门针对 PageObject 的 Python 库 Page Objects。使用 Page Objects 可以迅速实现 PageObject 模式，下面来看下 Page Objects 库的使用。</p>
<ul data-nodeid="84699">
<li data-nodeid="84700">
<p data-nodeid="84701"><strong data-nodeid="84871">安装：</strong></p>
</li>
</ul>
<pre class="lang-python" data-nodeid="84702"><code data-language="python">pip install page_objects
</code></pre>
<ul data-nodeid="84703">
<li data-nodeid="84704">
<p data-nodeid="84705"><strong data-nodeid="84875">应用：</strong></p>
</li>
</ul>
<pre class="lang-python" data-nodeid="84706"><code data-language="python"><span class="hljs-comment"># 以下为官方示例</span>
<span class="hljs-meta">&gt;&gt;&gt; </span><span class="hljs-keyword">from</span> page_objects <span class="hljs-keyword">import</span> PageObject, PageElement
<span class="hljs-meta">&gt;&gt;&gt; </span><span class="hljs-keyword">from</span> selenium <span class="hljs-keyword">import</span> webdriver
<span class="hljs-meta">&gt;&gt;&gt; </span><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LoginPage</span>(<span class="hljs-params">PageObject</span>):</span>
        username = PageElement(id_=<span class="hljs-string">'username'</span>)
        password = PageElement(name=<span class="hljs-string">'password'</span>)
        login = PageElement(css=<span class="hljs-string">'input[type="submit"]'</span>)
<span class="hljs-meta">&gt;&gt;&gt; </span>driver = webdriver.PhantomJS()
<span class="hljs-meta">&gt;&gt;&gt; </span>driver.get(<span class="hljs-string">"http://example.com"</span>)
<span class="hljs-meta">&gt;&gt;&gt; </span>page = LoginPage(driver)
<span class="hljs-meta">&gt;&gt;&gt; </span>page.username = <span class="hljs-string">'secret'</span>
<span class="hljs-meta">&gt;&gt;&gt; </span>page.password = <span class="hljs-string">'squirrel'</span>
<span class="hljs-meta">&gt;&gt;&gt; </span><span class="hljs-keyword">assert</span> page.username.text == <span class="hljs-string">'secret'</span>
<span class="hljs-meta">&gt;&gt;&gt; </span>page.login.click()
</code></pre>
<h3 data-nodeid="84707">项目实战 —— PageObject 应用</h3>
<p data-nodeid="84708">好，我们不仅了解了 PageObject 的理论，也了解了 page_objects 这个 Python 库的使用。</p>
<p data-nodeid="84709">现在，我们给我们的项目应用 pageObject 模型。</p>
<h4 data-nodeid="84710">第一步：改造项目结构</h4>
<p data-nodeid="84711">我们来按照 PageObject 的实现来改造我们的项目结构。改造前，我们的目录结构：</p>
<pre class="lang-python" data-nodeid="84712"><code data-language="python">|--lagouAPITest
    |--tests
        |--test_ones.py
        |--__init__.py
    |--common
        |--__init__.py
</code></pre>
<p data-nodeid="84713">其中，test_ones.py 是我们上一节课**《11| 如虎添翼，API 和 UI 自动化测试融合》**新建的文件，其余目录及 <strong data-nodeid="84902">init</strong>.py 都是空目录、空文件。<br>
使用 PageObject 改造后，我们期望的目录结构：</p>
<pre class="lang-python" data-nodeid="84714"><code data-language="python">|--lagouAPITest
    |--pages
        |--ones.py
        |--base_page.py
    |--tests
        |--test_ones.py
        |--__init__.py
    |--common
        |--__init__.py
</code></pre>
<p data-nodeid="84715">改造后，我们把原本的 tests 目录下的 test_ones.py 里关于页面元素的操作全部剥离到 pages 文件夹下的 ones.py 里，然后针对可能的公用的操作，我们进一步抽象到 base_page.py 里去。</p>
<p data-nodeid="84716">下面先看下原来 tests 文件夹下 test_ones.py 的内容：</p>
<pre class="lang-python" data-nodeid="84717"><code data-language="python"><span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-keyword">import</span> json
<span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">import</span> pytest
<span class="hljs-keyword">from</span> selenium <span class="hljs-keyword">import</span> webdriver
<span class="hljs-keyword">from</span> selenium.webdriver.common.by <span class="hljs-keyword">import</span> By
<span class="hljs-keyword">from</span> selenium.webdriver.support.ui <span class="hljs-keyword">import</span> WebDriverWait
<span class="hljs-keyword">from</span> selenium.webdriver.support <span class="hljs-keyword">import</span> expected_conditions <span class="hljs-keyword">as</span> EC

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">cookie_to_selenium_format</span>(<span class="hljs-params">cookie</span>):</span>
    cookie_selenium_mapping = {<span class="hljs-string">'path'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'secure'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'value'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'expires'</span>: <span class="hljs-string">''</span>}
    cookie_dict = {}
    <span class="hljs-keyword">if</span> getattr(cookie, <span class="hljs-string">'domain_initial_dot'</span>):
        cookie_dict[<span class="hljs-string">'domain'</span>] = <span class="hljs-string">'.'</span> + getattr(cookie, <span class="hljs-string">'domain'</span>)
    <span class="hljs-keyword">else</span>:
        cookie_dict[<span class="hljs-string">'domain'</span>] = getattr(cookie, <span class="hljs-string">'domain'</span>)
    <span class="hljs-keyword">for</span> k <span class="hljs-keyword">in</span> list(cookie_selenium_mapping.keys()):
        key = k
        value = getattr(cookie, k)
        cookie_dict[key] = value
    <span class="hljs-keyword">return</span> cookie_dict

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TestOneAI</span>:</span>
    <span class="hljs-comment"># 在pytest里，针对一个类方法的setup为setup_method,</span>
    <span class="hljs-comment"># setup_method作用同unittest里的setUp()</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">setup_method</span>(<span class="hljs-params">self, method</span>):</span>
        self.s = requests.Session()
        self.login_url = <span class="hljs-string">'https://ones.ai/project/api/project/auth/login'</span>
        self.home_page = <span class="hljs-string">'https://ones.ai/project/#/home/project'</span>
        self.header = {
            <span class="hljs-string">"user-agent"</span>: <span class="hljs-string">"user-agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"</span>,
            <span class="hljs-string">"content-type"</span>: <span class="hljs-string">"application/json"</span>}
        self.driver = webdriver.Chrome()
<span class="hljs-meta">    @pytest.mark.parametrize('login_data, project_name', [({"password": "iTestingIsGood", "email": "pleasefollowiTesting@outlook.com"}, {"project_name":"VIPTEST"})])</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test_merge_api_ui</span>(<span class="hljs-params">self, login_data, project_name</span>):</span>
        result = self.s.post(self.login_url, data=json.dumps(login_data), headers=self.header)
        <span class="hljs-keyword">assert</span> result.status_code == <span class="hljs-number">200</span>
        <span class="hljs-keyword">assert</span> json.loads(result.text)[<span class="hljs-string">"user"</span>][<span class="hljs-string">"email"</span>].lower() == login_data[<span class="hljs-string">"email"</span>]
        all_cookies = self.s.cookies._cookies[<span class="hljs-string">".ones.ai"</span>][<span class="hljs-string">"/"</span>]
        self.driver.get(self.home_page)
        self.driver.delete_all_cookies()
        <span class="hljs-keyword">for</span> k, v <span class="hljs-keyword">in</span> all_cookies.items():
            print(v)
            print(type(v))
            self.driver.add_cookie(cookie_to_selenium_format(v))
        self.driver.get(self.home_page)
        <span class="hljs-keyword">try</span>:
            element = WebDriverWait(self.driver, <span class="hljs-number">30</span>).until(
                EC.presence_of_element_located((By.CSS_SELECTOR, <span class="hljs-string">'[class="company-title-text"]'</span>)))
            <span class="hljs-keyword">assert</span> element.get_attribute(<span class="hljs-string">"innerHTML"</span>) == project_name[<span class="hljs-string">"project_name"</span>]
        <span class="hljs-keyword">except</span> TimeoutError:
            <span class="hljs-keyword">raise</span> TimeoutError(<span class="hljs-string">'Run time out'</span>)
    <span class="hljs-comment"># 在pytest里，针对一个类方法的teardown为teardown_method,</span>
    <span class="hljs-comment"># teardown_method作用同unittest里的dearDown()</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">teardown_method</span>(<span class="hljs-params">self, method</span>):</span>
        self.s.close()
        self.driver.quit()
</code></pre>
<h4 data-nodeid="84718">第二步，创建 Page 类</h4>
<p data-nodeid="84719">首先把跟页面有关的全部操作都放到 Page 类里，pages/ones.py 的代码如下：</p>
<pre class="lang-python" data-nodeid="84720"><code data-language="python"><span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-keyword">import</span> json
<span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">from</span> selenium <span class="hljs-keyword">import</span> webdriver
<span class="hljs-keyword">from</span> selenium.webdriver.common.by <span class="hljs-keyword">import</span> By
<span class="hljs-keyword">from</span> selenium.webdriver.support.ui <span class="hljs-keyword">import</span> WebDriverWait
<span class="hljs-keyword">from</span> selenium.webdriver.support <span class="hljs-keyword">import</span> expected_conditions <span class="hljs-keyword">as</span> EC
<span class="hljs-keyword">from</span> page_objects <span class="hljs-keyword">import</span> PageObject, PageElement

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">cookie_to_selenium_format</span>(<span class="hljs-params">cookie</span>):</span>
    cookie_selenium_mapping = {<span class="hljs-string">'path'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'secure'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'value'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'expires'</span>: <span class="hljs-string">''</span>}
    cookie_dict = {}
    <span class="hljs-keyword">if</span> getattr(cookie, <span class="hljs-string">'domain_initial_dot'</span>):
        cookie_dict[<span class="hljs-string">'domain'</span>] = <span class="hljs-string">'.'</span> + getattr(cookie, <span class="hljs-string">'domain'</span>)
    <span class="hljs-keyword">else</span>:
        cookie_dict[<span class="hljs-string">'domain'</span>] = getattr(cookie, <span class="hljs-string">'domain'</span>)
    <span class="hljs-keyword">for</span> k <span class="hljs-keyword">in</span> list(cookie_selenium_mapping.keys()):
        key = k
        value = getattr(cookie, k)
        cookie_dict[key] = value
    <span class="hljs-keyword">return</span> cookie_dict

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OneAI</span>(<span class="hljs-params">PageObject</span>):</span>
    <span class="hljs-comment"># 使用page_objects库把元素locator， 元素定位，元素操作分离</span>
    <span class="hljs-comment"># 元素定位的字符串</span>
    PROJECT_NAME_LOCATOR = <span class="hljs-string">'[class="company-title-text"]'</span>
    NEW_PROJECT_LOCATOR = <span class="hljs-string">'.ones-btn.ones-btn-primary'</span>

    <span class="hljs-comment"># 元素定位</span>
    new_project = PageElement(css=NEW_PROJECT_LOCATOR)
    <span class="hljs-comment"># 通过构造函数初始化浏览器driver，requests.Session()</span>
    <span class="hljs-comment"># 通过api_login方法直接带登录态到达待测试页面开始测试</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-params">self, login_credential, target_page</span>):</span>
        self.login_url = <span class="hljs-string">'https://ones.ai/project/api/project/auth/login'</span>
        self.header = {
            <span class="hljs-string">"user-agent"</span>: <span class="hljs-string">"user-agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"</span>,
            <span class="hljs-string">"content-type"</span>: <span class="hljs-string">"application/json"</span>}
        self.s = requests.Session()
        self.driver = webdriver.Chrome()
        self.api_login(login_credential, target_page)
    <span class="hljs-comment"># 融合API测试和UI测试，并传递登录态到浏览器Driver供使用</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">api_login</span>(<span class="hljs-params">self, login_credential, target_page</span>):</span>
        target_url = json.loads(json.dumps(target_page))
        <span class="hljs-keyword">try</span>:
            result = self.s.post(self.login_url, data=json.dumps(login_credential), headers=self.header)
            <span class="hljs-keyword">assert</span> result.status_code == <span class="hljs-number">200</span>
            <span class="hljs-keyword">assert</span> json.loads(result.text)[<span class="hljs-string">"user"</span>][<span class="hljs-string">"email"</span>].lower() == login_credential[<span class="hljs-string">"email"</span>]
        <span class="hljs-keyword">except</span> Exception:
            <span class="hljs-keyword">raise</span> Exception(<span class="hljs-string">"Login Failed, please check!"</span>)
        all_cookies = self.s.cookies._cookies[<span class="hljs-string">".ones.ai"</span>][<span class="hljs-string">"/"</span>]
        self.driver.get(target_url[<span class="hljs-string">"target_page"</span>])
        self.driver.delete_all_cookies()
        <span class="hljs-keyword">for</span> k, v <span class="hljs-keyword">in</span> all_cookies.items():
            self.driver.add_cookie(cookie_to_selenium_format(v))
        self.driver.get(target_url[<span class="hljs-string">"target_page"</span>])
        <span class="hljs-keyword">return</span> self.driver
    <span class="hljs-comment"># 功能函数</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_project_name</span>(<span class="hljs-params">self</span>):</span>
        <span class="hljs-keyword">try</span>:
            project_name = WebDriverWait(self.driver, <span class="hljs-number">30</span>).until(
                EC.presence_of_element_located((By.CSS_SELECTOR, self.PROJECT_NAME_LOCATOR)))
            <span class="hljs-keyword">return</span> project_name.get_attribute(<span class="hljs-string">"innerHTML"</span>)
        <span class="hljs-keyword">except</span> TimeoutError:
            <span class="hljs-keyword">raise</span> TimeoutError(<span class="hljs-string">'Run time out'</span>)
</code></pre>
<p data-nodeid="84721">我们来看下这个 ones.py 文件：</p>
<ul data-nodeid="84722">
<li data-nodeid="84723">
<p data-nodeid="84724">首先，我在其中定义了一个方法 cookie_to_selenium_format，这个方法是把通过 requests.Session() 拿到的 cookies 转换成 Selenium/WebDrvier 认可的格式，这个函数跟我们的 Page 类无关，所以我把它放在 Page 类外；</p>
</li>
<li data-nodeid="84725">
<p data-nodeid="84726">接着，我定义了 OneAI 这个 Page 类，并且按照 page_objects 这个库的推荐写法写了元素的定位。注意，我把元素的 Locator 本身，以及元素、元素操作都分离开了。这样当有任意一个修改时，都不影响另外两个；</p>
</li>
<li data-nodeid="84727">
<p data-nodeid="84728">然后我又写了类方法，一个是用于拿登录态直接通过浏览器访问页面的函数 api_login，还有一个就是获取 project_name 的文本的函数 get_project_name。</p>
</li>
</ul>
<p data-nodeid="84729">至此，我的第一版 Page 类就创建完毕，<strong data-nodeid="84937">但注意我这个 Page 类里是不包括测试的方法的。</strong></p>
<h4 data-nodeid="84730">第三步， 更新 TestPage 类</h4>
<p data-nodeid="84731">Page 类创建好，我们就要创建 Page 类对应的测试类。更改 tests 文件夹下的 test_ones.py 文件，更改后的内容如下：</p>
<pre class="lang-python" data-nodeid="84732"><code data-language="python"><span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-keyword">import</span> pytest
<span class="hljs-keyword">from</span> pages.ones <span class="hljs-keyword">import</span> OneAI

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TestOneAI</span>:</span>
    <span class="hljs-comment"># 注意，需要email和密码需要更改成你自己的账户密码</span>
<span class="hljs-meta">    @pytest.mark.parametrize('login_data, project_name, target_page', [({"password": "iTestingIsGood", "email": "pleasefollowiTesting@outlook.com"}, {"project_name":"VIPTEST"}, {"target_page": "https://ones.ai/project/#/home/project"})])</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test_project_name_txt</span>(<span class="hljs-params">self, login_data, project_name, target_page</span>):</span>
        print(login_data)
        one_page = OneAI(login_data, target_page)
        actual_project_name = one_page.get_project_name()
        <span class="hljs-keyword">assert</span> actual_project_name == project_name[<span class="hljs-string">"project_name"</span>]
</code></pre>
<p data-nodeid="84733">你可以看到，这个测试类就变得非常简洁。它只包括一个测试方法，即 test_project_name_txt。这个函数用来测试我们拿到的 project name 是不是等于我们提供的那个值，即 VIPTEST。</p>
<p data-nodeid="84734"><strong data-nodeid="84952">你需要注意，在测试类中，应该仅仅包括对 Page 类的各种方法的调用，而不能在测试类中直接去操作测试类对象生成新的功能。</strong></p>
<p data-nodeid="84735">我们在命令行运行下，输入以下命令：</p>
<pre class="lang-python" data-nodeid="84736"><code data-language="python">D:\_Automation\lagouAPITest&gt;pytest tests/test_ones.py
</code></pre>
<p data-nodeid="84737">测试运行结束后，查看运行结果如下：</p>
<p data-nodeid="84738"><img src="https://s0.lgstatic.com/i/image/M00/60/82/Ciqc1F-NcJuAfRgwAAC13CMIBCs725.png" alt="Drawing 2.png" data-nodeid="84957"></p>
<p data-nodeid="84739">至此，Page 类和 TestPage 类的解耦已经完成。</p>
<h4 data-nodeid="84740">第四步， 提炼 BasePage 类</h4>
<p data-nodeid="84741">至此，PageObject 模式我们已经应用到我们的项目中了，不过你发现没有，我们的 Page 类里还有很多可以优化的地方，比如 cookie_to_selenium_format 这个方法，它不属于某一个具体的 Page 类，但又可以被多个 Page 类调用。</p>
<p data-nodeid="84742">再比如，初始化浏览器 Driver 的代码，和初始化 requests.Session() 的代码也不属于某一个具体的 Page，但是我们把它放入了 Page 里，所以，我们继续优化，在 pages 文件夹下创建 base_page 类，把跟 page 无关的操作都提炼出来。</p>
<p data-nodeid="84743">pages/base_page.py 文件的内容如下：</p>
<pre class="lang-python" data-nodeid="84744"><code data-language="python"><span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-keyword">import</span> json
<span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">from</span> selenium <span class="hljs-keyword">import</span> webdriver
<span class="hljs-keyword">from</span> page_objects <span class="hljs-keyword">import</span> PageObject, PageElement

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">cookie_to_selenium_format</span>(<span class="hljs-params">cookie</span>):</span>
    cookie_selenium_mapping = {<span class="hljs-string">'path'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'secure'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'value'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'expires'</span>: <span class="hljs-string">''</span>}
    cookie_dict = {}
    <span class="hljs-keyword">if</span> getattr(cookie, <span class="hljs-string">'domain_initial_dot'</span>):
        cookie_dict[<span class="hljs-string">'domain'</span>] = <span class="hljs-string">'.'</span> + getattr(cookie, <span class="hljs-string">'domain'</span>)
    <span class="hljs-keyword">else</span>:
        cookie_dict[<span class="hljs-string">'domain'</span>] = getattr(cookie, <span class="hljs-string">'domain'</span>)
    <span class="hljs-keyword">for</span> k <span class="hljs-keyword">in</span> list(cookie_selenium_mapping.keys()):
        key = k
        value = getattr(cookie, k)
        cookie_dict[key] = value
    <span class="hljs-keyword">return</span> cookie_dict

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BasePage</span>(<span class="hljs-params">PageObject</span>):</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-params">self, login_credential, target_page</span>):</span>
        self.login_url = <span class="hljs-string">'https://ones.ai/project/api/project/auth/login'</span>
        self.header = {
            <span class="hljs-string">"user-agent"</span>: <span class="hljs-string">"user-agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"</span>,
            <span class="hljs-string">"content-type"</span>: <span class="hljs-string">"application/json"</span>}
        self.s = requests.Session()
        self.driver = webdriver.Chrome()
        self._api_login(login_credential, target_page)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">_api_login</span>(<span class="hljs-params">self, login_credential, target_page</span>):</span>
        target_url = json.loads(json.dumps(target_page))
        <span class="hljs-keyword">try</span>:
            result = self.s.post(self.login_url, data=json.dumps(login_credential), headers=self.header)
            <span class="hljs-keyword">assert</span> result.status_code == <span class="hljs-number">200</span>
            <span class="hljs-keyword">assert</span> json.loads(result.text)[<span class="hljs-string">"user"</span>][<span class="hljs-string">"email"</span>].lower() == login_credential[<span class="hljs-string">"email"</span>]
        <span class="hljs-keyword">except</span> Exception:
            <span class="hljs-keyword">raise</span> Exception(<span class="hljs-string">"Login Failed, please check!"</span>)
        all_cookies = self.s.cookies._cookies[<span class="hljs-string">".ones.ai"</span>][<span class="hljs-string">"/"</span>]
        self.driver.get(target_url[<span class="hljs-string">"target_page"</span>])
        self.driver.delete_all_cookies()
        <span class="hljs-keyword">for</span> k, v <span class="hljs-keyword">in</span> all_cookies.items():
            self.driver.add_cookie(cookie_to_selenium_format(v))
        self.driver.get(target_url[<span class="hljs-string">"target_page"</span>])
        <span class="hljs-keyword">return</span> self.driver
</code></pre>
<p data-nodeid="84745">在 BasePage 这个类里，我把跟具体的某一个 page 的操作都剔除掉，仅仅留下共用的部分，比如初始化浏览器 driver、初始化 requests.Session()，然后我把用于登录后传递登录态的方法 api_login 改成一个类保护方法_api_login（即只允许 BasePage 的类实例和它的子类实例能访问_api_login 方法）。</p>
<p data-nodeid="84746">这个时候，我们的 pages 文件夹下的 ones.py 也要做相应更改，更新后的 pages/ones.py 的内容如下：</p>
<pre class="lang-python" data-nodeid="84747"><code data-language="python"><span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-keyword">from</span> selenium.webdriver.common.by <span class="hljs-keyword">import</span> By
<span class="hljs-keyword">from</span> selenium.webdriver.support.ui <span class="hljs-keyword">import</span> WebDriverWait
<span class="hljs-keyword">from</span> selenium.webdriver.support <span class="hljs-keyword">import</span> expected_conditions <span class="hljs-keyword">as</span> EC
<span class="hljs-keyword">from</span> page_objects <span class="hljs-keyword">import</span> PageObject, PageElement
<span class="hljs-keyword">from</span> pages.base_page <span class="hljs-keyword">import</span> BasePage

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OneAI</span>(<span class="hljs-params">BasePage</span>):</span>
    PROJECT_NAME_LOCATOR = <span class="hljs-string">'[class="company-title-text"]'</span>
    NEW_PROJECT_LOCATOR = <span class="hljs-string">'.ones-btn.ones-btn-primary'</span>
    new_project = PageElement(css=NEW_PROJECT_LOCATOR)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-params">self, login_credential, target_page</span>):</span>
        super().__init__(login_credential, target_page)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_project_name</span>(<span class="hljs-params">self</span>):</span>
        <span class="hljs-keyword">try</span>:
            project_name = WebDriverWait(self.driver, <span class="hljs-number">30</span>).until(
                EC.presence_of_element_located((By.CSS_SELECTOR, self.PROJECT_NAME_LOCATOR)))
            <span class="hljs-keyword">return</span> project_name.get_attribute(<span class="hljs-string">"innerHTML"</span>)
        <span class="hljs-keyword">except</span> TimeoutError:
            <span class="hljs-keyword">raise</span> TimeoutError(<span class="hljs-string">'Run time out'</span>)
</code></pre>
<p data-nodeid="84748">可以看到。我们的 Page 类进一步简化，只包括 Page 本身的元素、对象和操作，而不包括其他的部分，比如对浏览器 Driver 的初始化、对 requests.Session() 的初始化、登录等操作了。</p>
<p data-nodeid="84749">这个时候再回到《02 | 反复践行的 13 条自动化测试框架设计原则》里最初提问的那个问题，当你有了新的项目，你老板会说：“你不是有框架吗？快上个自动化”，这个时候你要怎么回答？</p>
<p data-nodeid="84750">学习了 Page Object 模型后的你这时候是不是可以一边心里窃喜，一边脸上还要装作工作量很大很为难的样子，说“好的老板我尽量这周给到”。</p>
<p data-nodeid="84751">实际上，你只要把 Pages 和 Tests 这两个文件夹下的文件删除，换成你新项目的文件就好了。所以，你说 Page Object 模型好不好啊？根据《02 | 反复践行的 13 条自动化测试框架设计原则》的指导来设计测试框架，是不是事半功倍啊！</p>
<h4 data-nodeid="84752">第五步， 打造通用性测试框架</h4>
<p data-nodeid="84753">好了，本节课到现在，我们已经把 PageObject 模式的应用全部掌握了。现在来看看我们的框架，你觉得还有改进的空间吗？</p>
<pre class="lang-python" data-nodeid="84754"><code data-language="python">|--lagouAPITest
    |--pages
        |--ones.py
        |--base_page.py
    |--tests
        |--test_ones.py
        |--__init__.py
    |--common
        |--__init__.py
</code></pre>
<p data-nodeid="84755">当然有改进空间了， 再看看 base_page.py 这个文件。既然是 base_page，那么只应该跟 page 有关系，可是我们把初始化浏览器 driver、初始化 requests.Session() 这样的操作也放进去了，是不是不太合理？还有万一以后的浏览器测试不用 Selenium/WebDriver 了呢？</p>
<blockquote data-nodeid="84756">
<p data-nodeid="84757">我个人估计这个进程会很快，现在前端自动化这边 Cypress 发展迅猛，而且上手非常简单。一个人通过简单学习很快就能通过 Cypress 来搭建基于持续集成的自动化测试框架。关于 Cypress，你可以参考我的新书《前端自动化测试框架 Cypress 从入门到精通》。</p>
</blockquote>
<p data-nodeid="84758">那么假设以后我们的浏览器测试不用Selenium/webDriver了，还有，万一有比 requests 更好用的HTTP库了呢？所以，有必要进一步拆分。</p>
<p data-nodeid="84759">于是，我们的框架结构就变成如下的样子：</p>
<pre class="lang-python" data-nodeid="84760"><code data-language="python">|--lagouAPITest
    |--pages
        |--ones.py
        |--base_page.py
    |--tests
        |--test_ones.py
        |--__init__.py
    |--common
        |--__init__.py
        |--selenium_helper.py
        |--requests_helper.py
</code></pre>
<p data-nodeid="84761">把 BasePage 这个类里的关于 Selenium/WebDriver 和 Requests 的操作分别拆分到 selenium_helper.py 里和 requests_helper.py 里去。</p>
<p data-nodeid="84762">selenium_helper.py 的内容如下：</p>
<pre class="lang-python" data-nodeid="84763"><code data-language="python">__author__ = <span class="hljs-string">'kevin'</span>
<span class="hljs-keyword">from</span> selenium <span class="hljs-keyword">import</span> webdriver

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SeleniumHelper</span>(<span class="hljs-params">object</span>):</span>
<span class="hljs-meta">    @staticmethod</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">initial_driver</span>(<span class="hljs-params">browser_name=<span class="hljs-string">'chrome'</span></span>):</span>
        browser_name = browser_name.lower()
        <span class="hljs-keyword">if</span> browser_name <span class="hljs-keyword">not</span> <span class="hljs-keyword">in</span> {<span class="hljs-string">'chrome'</span>, <span class="hljs-string">'firefox'</span>, <span class="hljs-string">'ff'</span>, <span class="hljs-string">'ie'</span>}:
            browser_name = <span class="hljs-string">'chrome'</span>
        <span class="hljs-keyword">if</span> browser_name == <span class="hljs-string">'chrome'</span>:
            browser = webdriver.Chrome()
        <span class="hljs-keyword">elif</span> browser_name <span class="hljs-keyword">in</span> (<span class="hljs-string">'firefox'</span>, <span class="hljs-string">'ff'</span>):
            browser = webdriver.Firefox()
        <span class="hljs-keyword">elif</span> browser_name == <span class="hljs-string">'ie'</span>:
            webdriver.Ie()
        browser.maximize_window()
        browser.implicitly_wait(<span class="hljs-number">60</span>)
        <span class="hljs-keyword">return</span> browser
<span class="hljs-meta">    @staticmethod</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">cookie_to_selenium_format</span>(<span class="hljs-params">cookie</span>):</span>
        cookie_selenium_mapping = {<span class="hljs-string">'path'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'secure'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'value'</span>: <span class="hljs-string">''</span>, <span class="hljs-string">'expires'</span>: <span class="hljs-string">''</span>}
        cookie_dict = {}
        <span class="hljs-keyword">if</span> getattr(cookie, <span class="hljs-string">'domain_initial_dot'</span>):
            cookie_dict[<span class="hljs-string">'domain'</span>] = <span class="hljs-string">'.'</span> + getattr(cookie, <span class="hljs-string">'domain'</span>)
        <span class="hljs-keyword">else</span>:
            cookie_dict[<span class="hljs-string">'domain'</span>] = getattr(cookie, <span class="hljs-string">'domain'</span>)
        <span class="hljs-keyword">for</span> k <span class="hljs-keyword">in</span> list(cookie_selenium_mapping.keys()):
            key = k
            value = getattr(cookie, k)
            cookie_dict[key] = value
        <span class="hljs-keyword">return</span> cookie_dict
</code></pre>
<p data-nodeid="84764">selenium_helper.py 里包括了所有针对 Selenium 的操作，以后针对浏览器的各种操作全部都放在这个文件。</p>
<p data-nodeid="84765">requests_helper.py 里的代码，更新后如下：</p>
<pre class="lang-python" data-nodeid="84766"><code data-language="python"><span class="hljs-keyword">import</span> json
<span class="hljs-keyword">import</span> traceback
<span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">from</span> requests.packages.urllib3.exceptions <span class="hljs-keyword">import</span> InsecureRequestWarning
<span class="hljs-comment"># Disable https security warning</span>
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SharedAPI</span>(<span class="hljs-params">object</span>):</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-params">self</span>):</span>
        self.s = requests.session()
        self.login_url = <span class="hljs-string">'https://ones.ai/project/api/project/auth/login'</span>
        self.header = {
            <span class="hljs-string">"user-agent"</span>: <span class="hljs-string">"user-agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"</span>,
            <span class="hljs-string">"content-type"</span>: <span class="hljs-string">"application/json"</span>}
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">login</span>(<span class="hljs-params">self, login_credential</span>):</span>
        <span class="hljs-keyword">try</span>:
            result = self.s.post(self.login_url, data=json.dumps(login_credential), headers=self.header, verify=<span class="hljs-literal">False</span>)
            <span class="hljs-keyword">if</span> int(result.status_code) == <span class="hljs-number">200</span>:
                <span class="hljs-keyword">pass</span>
            <span class="hljs-keyword">else</span>:
                <span class="hljs-keyword">raise</span> Exception(<span class="hljs-string">'login failed'</span>)
            <span class="hljs-keyword">return</span> result
        <span class="hljs-keyword">except</span> RuntimeError:
            traceback.print_exc()
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">post_api</span>(<span class="hljs-params">self, url, **kwargs</span>):</span>
        <span class="hljs-keyword">return</span> self.s.post(url, **kwargs)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_api</span>(<span class="hljs-params">self, url, **kwargs</span>):</span>
        <span class="hljs-keyword">return</span> self.s.get(url, **kwargs)
</code></pre>
<p data-nodeid="84767">requests_helper.py 里包括所有对 requests 这个库的操作。</p>
<p data-nodeid="84768">最后，我们还需要更新下 base_page.py，base_page.py 更新后，内容如下：</p>
<pre class="lang-python" data-nodeid="84769"><code data-language="python"><span class="hljs-comment"># -*- coding: utf-8 -*-</span>
<span class="hljs-keyword">import</span> json
<span class="hljs-keyword">import</span> traceback
<span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">from</span> selenium <span class="hljs-keyword">import</span> webdriver
<span class="hljs-keyword">from</span> page_objects <span class="hljs-keyword">import</span> PageObject, PageElement
<span class="hljs-keyword">from</span> common.requests_helper <span class="hljs-keyword">import</span> SharedAPI
<span class="hljs-keyword">from</span> common.selenium_helper <span class="hljs-keyword">import</span> SeleniumHelper

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BasePage</span>(<span class="hljs-params">PageObject</span>):</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-params">self, login_credential, target_page</span>):</span>
        self.api_driver = SharedAPI()
        self.loginResult = self.api_driver.login(login_credential)
        self.driver = SeleniumHelper.initial_driver()
        self._api_login(login_credential, target_page)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">_api_login</span>(<span class="hljs-params">self, login_credential, target_page</span>):</span>
        target_url = json.loads(json.dumps(target_page))
        <span class="hljs-keyword">assert</span> json.loads(self.loginResult.text)[<span class="hljs-string">"user"</span>][<span class="hljs-string">"email"</span>].lower() == login_credential[<span class="hljs-string">"email"</span>]
        all_cookies = self.loginResult.cookies._cookies[<span class="hljs-string">".ones.ai"</span>][<span class="hljs-string">"/"</span>]
        self.driver.get(target_url[<span class="hljs-string">"target_page"</span>])
        self.driver.delete_all_cookies()
        <span class="hljs-keyword">for</span> k, v <span class="hljs-keyword">in</span> all_cookies.items():
            self.driver.add_cookie(SeleniumHelper.cookie_to_selenium_format(v))
        self.driver.get(target_url[<span class="hljs-string">"target_page"</span>])
        <span class="hljs-keyword">return</span> self.driver
</code></pre>
<p data-nodeid="84770">可以看到，在更新后的 base_page.py 里，我们初始化 requests.Session() 和浏览器的 Driver 的方式是通过调用 SharedAPI 和 SeleniumHelper 这两个类。然后 BasePage 这个类里现在只包括各个 Page 类可以共用的函数，而不再包括无关的操作。</p>
<p data-nodeid="84771">其他文件无须更改。</p>
<p data-nodeid="84772">通过如下命令运行：</p>
<pre class="lang-python" data-nodeid="84773"><code data-language="python">D:\_Automation\lagouAPITest&gt;pytest --alluredir=./allure_reports
</code></pre>
<p data-nodeid="84774">然后使用命令行进入到你的项目根目录下，执行如下语句：</p>
<pre class="lang-python" data-nodeid="84775"><code data-language="python">D:\_Automation\lagouAPITest&gt;allure serve allure_reports
</code></pre>
<p data-nodeid="84776">接着你的默认浏览器会自动打开测试报告，看下运行结果：</p>
<p data-nodeid="84777"><img src="https://s0.lgstatic.com/i/image/M00/60/82/Ciqc1F-NcfmAFQfWAABpixBpo-E991.png" alt="Drawing 3.png" data-nodeid="85034"></p>
<p data-nodeid="84778">好的，大功告成。</p>
<p data-nodeid="84779">你发现没有，通过这个方式，我们就把自动化测试框架进行了框架代码和业务代码的剥离。此后我们的框架不仅看起来结构清晰，而且也变得跟业务松耦合。当你需要在新项目应用自动化的时候，仅仅把 pages 和 tests 这两个文件夹更换，便能够一“秒”搭建新的测试框架。</p>
<p data-nodeid="84780">我们的《测试开发入门与实战》课程，直到这里才有了一点点测试开发的味道。迄今为止， 我们通过学习，已经能够搭建一套融合 API 测试和 UI 测试，并且具备现代测试框架雏形的框架。</p>
<h3 data-nodeid="84781">小结</h3>
<p data-nodeid="84782">本节课我主要讲解了 PageObject 模式，PageObject 模式是自动化测试里的一个经典设计模式。通过 PageObject 我们可以实现元素定位、元素、元素操作的分离，从而让自己的自动化测试框架更加具备可重用性。</p>
<p data-nodeid="84783">在实现 PageObject 的同时，我主要通过展现思维的方式，向你一步步讲解如何把一个什么都耦合在一起的测试框架，一点点地剥离开。</p>
<p data-nodeid="84784">在练习本节课的过程中，我希望你不仅仅是 copy 下代码执行，我希望你能多关注我在这一课时“<strong data-nodeid="85046">项目实战——PageObject 应用”这一小节</strong>中展示出来的思维过程，并在以后的学习中，刻意锻炼自己这种发现问题—解决问题的能力。</p>
<p data-nodeid="84785">唯有此，你才能在测试开发的道路上越走越远。</p>
<p data-nodeid="84786"><strong data-nodeid="85055">好了，本节课最后给大家布置一个课后作业：按照本节课所讲，把我们第 7、8 课时“你的第一个 Web 测试框架”学习的两个测试文件 test_baidu.py 和 test_lagou.py 应用至 PageObject 模型中。</strong></p>
<p data-nodeid="84787">最后，你会发现我们的项目就会从下面这个样子：</p>
<pre class="lang-python" data-nodeid="84788"><code data-language="python">|--lagouAPITest
    |--pages
        |--ones.py
        |--base_page.py
    |--tests
        |--test_ones.py
        |--__init__.py
    |--common
        |--__init__.py
        |--selenium_helper.py
        |--requests_helper.py
</code></pre>
<p data-nodeid="84789">变成：</p>
<pre class="lang-python" data-nodeid="84790"><code data-language="python">|--lagouAPITest
    |--pages
        |--baidu.py
        |--lagou.py
        |--ones.py
        |--base_page.py
    |--tests
        |--test_ones.py
        |--test_baidu.py
        |--test_lagou.py
        |--__init__.py
    |--common
        |--__init__.py
        |--selenium_helper.py
        |--requests_helper.py
</code></pre>
<p data-nodeid="88579" class="">好的，本节课就到这里，也别忘完成课后作业哦～<br>
我是蔡超，我们下节课再见，下节课我将带你走入数据驱动的世界。</p>







<p data-nodeid="90536">更多关于测试框架的知识，请关注测试公众号 iTesting， 并回复 “测试框架”查看。</p>
<hr data-nodeid="90537">
<p data-nodeid="90538"><a href="https://wj.qq.com/s2/7506053/9b01" data-nodeid="90542">课程评价入口，挑选 5 名小伙伴赠送小礼品～</a></p>

---

### 精选评论

##### John：
> self.login_url = 'https://ones.ai/project/api/project/auth/login'这个地址not found了，里面的邮箱和密码不能使用吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以自己注册下。这个不是重点，重点是理解下代码的逻辑。

##### **红：
> 追问下，那这setup_method和teardown_method是放在test_ones.py中吗？或者放在init.py?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; setup_method和teardown_method是测试的前置和后置，如果需要用到，放在测试类下，作为类成员函数就可以。

##### **红：
> 老师，teardown和setup放在哪里了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; setup_method， 以及teardown_method

##### **婷：
> 请问下class TestOneAI:    # 注意，需要email和密码需要更改成你自己的账户密码
    @pytest.mark.parametrize('login_data, project_name, target_page', [({"password": "iTestingIsGood", "email": "pleasefollowiTesting@outlook.com"}, {"project_name":"VIPTEST"}, {"target_page": "https://ones.ai/project/#/home/project"})])
    def test_project_name_txt(self, login_data, project_name, target_page):
        print(login_data)
        one_page = OneAI(login_data, target_page)以上最后一行OneAI(login_data, target_page),可是OneAI这个类的定义中，参数不是只有一个PageObject，如下class OneAI(PageObject):吗，还请教下是怎么回事。。。谢谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 参数是在__init__方法里定义的，在python中__init__方法就是构造函数。

##### William：
> 今天的内容很有分量，如果把今天的知识全部搞懂，代码全部自己电脑上能一行一行敲出来，感觉就像完成打怪副本直接升了一级！同学们一起加油

##### WINKY：
> requests_helper.py 里的代码里面的 de 是不是def post_api(self, url, **kwargs):

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哈哈是的，那个f被换到下一行了。 应该def在同一行。感谢指出！

##### **0967：
> requests_helper.py 模块里面的def post_api(self, url, **kwargs)代码有遗漏吧。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有遗漏,你是运行出错了吗？可以把具体的错误贴出来吗？我们一起看一下

