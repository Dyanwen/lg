<p data-nodeid="67296" class="">通过前面课时的讲解，我们已经对如何进行 UI 自动化和接口自动化测试有了相当深刻的理解。但是对于接口测试的分享，在前面课程的讲解中，我主要讲解了基于 HTTP 的 RESTFUL 的接口。</p>
<p data-nodeid="67297">实际上，接口有很多形式，除了我们常见的 HTTP 形式的 RESTFUL 接口外，还有 Web Services 类型的接口，以及 RPC 接口。不同类型的接口测试方式各有不同。</p>
<p data-nodeid="67298">今天我们就来看下，如何测试 Web Services 类型的接口，这节课的内容如下：</p>
<p data-nodeid="67299"><img src="https://s0.lgstatic.com/i/image/M00/6A/99/Ciqc1F-pCv6AaQg5AAYAaUPRT5M790.png" alt="图片9.png" data-nodeid="67405"></p>
<h3 data-nodeid="67300">什么是 Web Services？</h3>
<p data-nodeid="67301">Web Service 是一种跨编程语言和跨操作系统平台的远程调用技术。</p>
<p data-nodeid="67302">通俗一点说，Web Service 就是一个应用程序，它<strong data-nodeid="67413">通过向外界暴露一个能够通过 Web 进行调用的 API 来对外提供服务</strong>。WebService 可以跨编程语言和跨操作系统，即你的客户端程序和提供服务的服务端程序可以采用不同的编程语言，使用不同的操作系统。</p>
<p data-nodeid="67303">举个例子来说，通过 WebServices，你运行在 windows 平台上的、以 C++ 编写的客户端程序就可以和运行在 Linux 平台上的，以 Java 编写的服务器程序进行通信。</p>
<h3 data-nodeid="67304">Web Services 构成及调用原理</h3>
<p data-nodeid="67305">Web Service 平台的构成，依赖以下技术：</p>
<ul data-nodeid="67306">
<li data-nodeid="67307">
<p data-nodeid="67308"><strong data-nodeid="67421">UDDI</strong>意为统一描述、发现和集成（Universal Description, Discovery, and Integration）,它是一种目录服务，通过它企业可注册并搜索 Web services，它是基于 XML 的跨平台描述规范。</p>
</li>
<li data-nodeid="67309">
<p data-nodeid="67310"><strong data-nodeid="67426">SOAP</strong>是一种简单的基于 XML 的协议，它使应用程序通过 HTTP 来交换信息。</p>
</li>
<li data-nodeid="67311">
<p data-nodeid="67312"><strong data-nodeid="67431">WSDL</strong>是基于 XML 的，用于描述 Web Services，以及如何访问 Web Services 的语言。</p>
</li>
</ul>
<p data-nodeid="67313">Web service 的调用原理如下：</p>
<p data-nodeid="67314"><img src="https://s0.lgstatic.com/i/image/M00/6A/A3/CgqCHl-pCsSAD1CUAAFHQFINvZY722.png" alt="图片1.png" data-nodeid="67435"></p>
<ul data-nodeid="67315">
<li data-nodeid="67316">
<p data-nodeid="67317">Step 1. 客户端想调用一个服务，但是不知道去哪里调用，于是它向 UDDI 注册中心（UDDI Registry）询问；</p>
</li>
<li data-nodeid="67318">
<p data-nodeid="67319">Step 2. UDDI 注册中心，发现有个名字为 Web Service A 的服务器，可以提供客户端想要的服务；</p>
</li>
<li data-nodeid="67320">
<p data-nodeid="67321">Step 3. 客户端向 Web Service A 发送消息，询问应该如何调用它需要的服务；</p>
</li>
<li data-nodeid="67322">
<p data-nodeid="67323">Step 4. Web Service A 收到请求，发送给客户端一个 WSDL 文件。这里记录了 Web Service A 可以提供的各类方法接口；</p>
</li>
<li data-nodeid="67324">
<p data-nodeid="67325">Step 5. 客户端通过 WSDL 生成 SOAP 请求（将 Web Service 提供的 xml 格式的接口方法，采用 SOAP 协议封装成 HTTP 请求），发送给 Web Service A，调用它想要的服务；</p>
</li>
<li data-nodeid="67326">
<p data-nodeid="67327">Step 6. Web Service A 按照 SOAP 请求执行相应的服务，并将结果返回给客户端。</p>
</li>
</ul>
<h3 data-nodeid="67328">Web Services 接口和 API（应用程序接口）的区别</h3>
<p data-nodeid="67329">Web Services 接口和我们常用的 API（应用程序接口）有哪些区别呢？下面的表格展示了它们的区别：<br>
<img src="https://s0.lgstatic.com/i/image/M00/6A/A4/CgqCHl-pCtaAIlaHAAGumDmnrfI772.png" alt="图片3.png" data-nodeid="67447"></p>
<p data-nodeid="67330">在我们的日常工作中，接口是以 Web Service、API，还是 RESTFUL API 形式提供给我们测试，常常取决于业务的实际情况。</p>
<h3 data-nodeid="67331">Web Services 接口实战</h3>
<p data-nodeid="67332">通过前面的讲解我们了解，WSDL 是 Web Services 生成给客户端调用的接口服务描述。通过 WSDL，客户端就可以构造正确的请求发送给服务端。</p>
<p data-nodeid="67333">在实际工作中也是如此，对于 Web Services 形式的接口，开发提供的往往就是一个 WSDL 格式的链接。比如，下面的一个链接就是一个公用的 Web Servbice 服务接口：</p>
<pre class="lang-java" data-nodeid="67334"><code data-language="java"># IP地址服务
http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl
</code></pre>
<h4 data-nodeid="67335">1.suds - SOAP 客户端</h4>
<p data-nodeid="67336">在 Python 中，客户端调用 Web Service 可以通过 suds 这个库来实现。suds Client 类提供了用于使用 Web Service 的统一 API 对象，这个对象包括以下两个命名空间。</p>
<ul data-nodeid="67337">
<li data-nodeid="67338">
<p data-nodeid="67339">service：service 对象用来调用被消费的 web service 提供的方法。</p>
</li>
<li data-nodeid="67340">
<p data-nodeid="67341">factory：提供一个工厂（factory），可用于创建&nbsp;WSDL&nbsp;中定义的对象和类型的实例。</p>
</li>
</ul>
<p data-nodeid="67342">下面来具体讲解下 suds 的使用。</p>
<ul data-nodeid="67343">
<li data-nodeid="67344">
<p data-nodeid="67345"><strong data-nodeid="67460">suds 安装</strong></p>
</li>
</ul>
<p data-nodeid="67346">在 Python 官方停止支持 Python 2.X 版本并全面转到 Python 3.X 后，suds 原始项目的开发已经停滞了，但这不意味着 suds 不再支持 Python 3.X。suds-community fork 了原本的 suds 库，并开发了能够支持 Python 3.X的 版本，其安装也比较简单：</p>
<pre class="lang-java" data-nodeid="67347"><code data-language="java">pip install suds-community
</code></pre>
<ul data-nodeid="67348">
<li data-nodeid="67349">
<p data-nodeid="67350"><strong data-nodeid="67465">简单使用</strong></p>
</li>
</ul>
<pre class="lang-python" data-nodeid="67351"><code data-language="python"><span class="hljs-keyword">from</span> suds.client <span class="hljs-keyword">import</span> Client
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    url = <span class="hljs-string">'http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl'</span>
    <span class="hljs-comment"># 初始化</span>
    client = Client(url)
    <span class="hljs-comment"># 打印出所有可用的方法</span>
    print(client)
</code></pre>
<p data-nodeid="67352">直接运行上述代码，你会发现执行结果如下：</p>
<pre class="lang-python" data-nodeid="67353"><code data-language="python"><span class="hljs-comment"># 运行结果片段</span>
Suds ( https://fedorahosted.org/suds/ )&nbsp; version: <span class="hljs-number">0.8</span><span class="hljs-number">.4</span>
Service ( IpAddressSearchWebService ) tns=<span class="hljs-string">"http://WebXml.com.cn/"</span>
&nbsp; &nbsp;Prefixes (<span class="hljs-number">1</span>)
&nbsp; &nbsp; &nbsp; ns0 = <span class="hljs-string">"http://WebXml.com.cn/"</span>
&nbsp; &nbsp;Ports (<span class="hljs-number">2</span>):
&nbsp; &nbsp; &nbsp; (IpAddressSearchWebServiceSoap)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Methods (<span class="hljs-number">3</span>):
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; getCountryCityByIp(xs:string theIpAddress)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; getGeoIPContext()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; getVersionTime()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Types (<span class="hljs-number">1</span>):
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ArrayOfString
&nbsp; &nbsp; &nbsp; (IpAddressSearchWebServiceSoap12)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Methods (<span class="hljs-number">3</span>):
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; getCountryCityByIp(xs:string theIpAddress)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; getGeoIPContext()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; getVersionTime()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Types (<span class="hljs-number">1</span>):
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ArrayOfString
</code></pre>
<p data-nodeid="67354">在这段代码中，我打印出来了 IpAddressSearchWebService 支持的所有方法。你可以看到， 它有三个方法（Methods(3) 显示出这个 Web Service 所提供的方法及参数）。</p>
<ul data-nodeid="67355">
<li data-nodeid="67356">
<p data-nodeid="67357"><strong data-nodeid="67471">实际案例</strong></p>
</li>
</ul>
<p data-nodeid="67358">既然看出 IpAddressSearchWebService 这个 Web Service 支持 3 种方法，那么我们来应用下这些方法：</p>
<pre class="lang-python" data-nodeid="67359"><code data-language="python"><span class="hljs-keyword">from</span> suds.client <span class="hljs-keyword">import</span> Client
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    url = <span class="hljs-string">'http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl'</span>
    <span class="hljs-comment"># 初始化</span>
    client = Client(url)
    <span class="hljs-comment"># 打印出所有支持的方法</span>
    print(client)
    <span class="hljs-comment"># 调用支持的方法， 使用client.service</span>
    print(client.service.getVersionTime())
    print(client.service.getCountryCityByIp(<span class="hljs-string">'192.168.0.1'</span>))
</code></pre>
<p data-nodeid="67360">执行上述代码，你会发现有如下输出：</p>
<pre class="lang-python" data-nodeid="67361"><code data-language="python"><span class="hljs-comment"># 输出结果片段</span>
<span class="hljs-comment">#此为getVersionTime这个方法的输出</span>
IP地址数据库，及时更新
<span class="hljs-comment"># 此为getCountryCityByIp方法的输出</span>
(ArrayOfString){
&nbsp; &nbsp;string[] =&nbsp;
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"192.168.0.1"</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"局域网 对方和您在同一内部网"</span>,
&nbsp;}
</code></pre>
<p data-nodeid="67362">注意，在代码里，我使用了 client.service 的方式，那是因为 service 对象用来调用被消费的 web service 提供的方法的。</p>
<p data-nodeid="67363">在实际工作中，你遇见的 WSDL 接口将会比这个复杂得多。故正常情况下，我们会将 WSDL 的接口封装成类使用，然后针对每个类方法，编写相应的测试用例，如下所示：</p>
<pre class="lang-python" data-nodeid="67364"><code data-language="python"><span class="hljs-keyword">import</span> pytest
<span class="hljs-keyword">from</span> suds.client <span class="hljs-keyword">import</span> Client

<span class="hljs-meta">@pytest.mark.rmb</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WebServices</span>(<span class="hljs-params">object</span>):</span>
    WSDL_ADDRESS = <span class="hljs-string">'http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl'</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-params">self</span>):</span>
        self.web_service = Client(self.WSDL_ADDRESS)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_version_time</span>(<span class="hljs-params">self</span>):</span>
        <span class="hljs-keyword">return</span> self.web_service.service.getVersionTime()
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_country_city_by_ip</span>(<span class="hljs-params">self, ip</span>):</span>
        <span class="hljs-keyword">return</span> self.web_service.service.getCountryCityByIp(ip)

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TestWebServices</span>:</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test_version_time</span>(<span class="hljs-params">self</span>):</span>
        <span class="hljs-keyword">assert</span> WebServices().get_version_time() == <span class="hljs-string">"IP地址数据库，及时更新"</span>
<span class="hljs-meta">    @pytest.mark.parametrize('ip, expected', [('10.10.10.10', '10.10.10.10')])</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test_get_country_city_by_ip</span>(<span class="hljs-params">self, ip, expected</span>):</span>
        <span class="hljs-keyword">assert</span> expected <span class="hljs-keyword">in</span> str(WebServices().get_country_city_by_ip(ip))

<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    pytest.main([<span class="hljs-string">"-m"</span>, <span class="hljs-string">"rmb"</span>, <span class="hljs-string">"-s"</span>, <span class="hljs-string">"-v"</span>])
</code></pre>
<h4 data-nodeid="67365">2.Zeep - SOAP 客户端</h4>
<p data-nodeid="67366">Zeep 是 Python 中的一个现代化的 SOAP 客户端。Zeep 通过检查 WSDL 文档并生成相应的代码，来使用 WSDL 文档中的服务和类型。这种方式为 SOAP 服务器提供了易于使用的编程接口。</p>
<p data-nodeid="67367">下面来具体讲解下 Zeep 的使用：</p>
<ul data-nodeid="67368">
<li data-nodeid="67369">
<p data-nodeid="67370"><strong data-nodeid="67482">Zeep 安装</strong></p>
</li>
</ul>
<pre class="lang-python" data-nodeid="67371"><code data-language="python">pip install zeep
</code></pre>
<ul data-nodeid="67372">
<li data-nodeid="67373">
<p data-nodeid="67374"><strong data-nodeid="67486">Zeep 查询 WSDL 中可用的方法</strong></p>
</li>
</ul>
<p data-nodeid="67375">相对于 suds 来说，想要查看一个 WSDL 描述中有哪些方法可用，Zeep 无须进行初始化动作。直接在命令行中输入如下命令即可：</p>
<pre class="lang-python" data-nodeid="67376"><code data-language="python">python -mzeep http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl
</code></pre>
<p data-nodeid="67377">执行后，会发现输出如下：</p>
<p data-nodeid="67378"><img src="https://s0.lgstatic.com/i/image/M00/6A/6D/CgqCHl-o-M2AMf3RAAC1Nxb0gQc472.png" alt="Drawing 2.png" data-nodeid="67491"></p>
<p data-nodeid="67379">从结果中可以看出，IpAddressSearchWebService 提供了 3 个 method，分别是&nbsp;getCountryCityByIp，getGeoIPContext 和 getVersionTime。</p>
<ul data-nodeid="67380">
<li data-nodeid="67381">
<p data-nodeid="67382"><strong data-nodeid="67496">简单使用</strong></p>
</li>
</ul>
<p data-nodeid="67383">在得出有哪些方法可用后，我们就可以像直接调用可用的方法：</p>
<pre class="lang-python" data-nodeid="67384"><code data-language="python"><span class="hljs-keyword">import</span> zeep
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    wsdl = <span class="hljs-string">'http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl'</span>
    client = zeep.Client(wsdl=wsdl)
    print(client.service.getCountryCityByIp(<span class="hljs-string">'10.10.10.10'</span>))
</code></pre>
<ul data-nodeid="67385">
<li data-nodeid="67386">
<p data-nodeid="67387"><strong data-nodeid="67501">实际案例</strong></p>
</li>
</ul>
<p data-nodeid="67388">现在我们把上面用 suds 实现的测试 IpAddressSearchWebService 的代码更改为使用 Zeep 测试：</p>
<pre class="lang-python" data-nodeid="67389"><code data-language="python"><span class="hljs-keyword">import</span> pytest
<span class="hljs-keyword">import</span> zeep
<span class="hljs-meta">@pytest.mark.rmb</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WebServices</span>(<span class="hljs-params">object</span>):</span>
    WSDL_ADDRESS = <span class="hljs-string">'http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl'</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-params">self</span>):</span>
        self.web_service = zeep.Client(wsdl=self.WSDL_ADDRESS)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_version_time</span>(<span class="hljs-params">self</span>):</span>
        <span class="hljs-keyword">return</span> self.web_service.service.getVersionTime()
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_country_city_by_ip</span>(<span class="hljs-params">self, ip</span>):</span>
        <span class="hljs-keyword">return</span> self.web_service.service.getCountryCityByIp(ip)

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TestWebServices</span>:</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test_version_time</span>(<span class="hljs-params">self</span>):</span>
        <span class="hljs-keyword">assert</span> WebServices().get_version_time() == <span class="hljs-string">"IP地址数据库，及时更新"</span>
<span class="hljs-meta">    @pytest.mark.parametrize('ip, expected', [('10.10.10.10', '10.10.10.10')])</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">test_get_country_city_by_ip</span>(<span class="hljs-params">self, ip, expected</span>):</span>
        <span class="hljs-keyword">assert</span> expected <span class="hljs-keyword">in</span> str(WebServices().get_country_city_by_ip(ip))

<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    pytest.main([<span class="hljs-string">"-m"</span>, <span class="hljs-string">"rmb"</span>, <span class="hljs-string">"-s"</span>, <span class="hljs-string">"-v"</span>])
</code></pre>
<p data-nodeid="67390">可以看到，使用 Zeep 来调用 Web Service 服务同样很简单。</p>
<h4 data-nodeid="67391">3.Zeep 和 suds 的比较</h4>
<p data-nodeid="67392">suds 是一个老牌的 SOAP 客户端，而 Zeep 是当前特别流行的一个 SOAP 客户端。那么我们应该如何选用呢？ 这里列出来几点两者的区别，供你参考：<br>
<img src="https://s0.lgstatic.com/i/image/M00/6A/99/Ciqc1F-pCu2AbMxCAAF3kfMvMmE625.png" alt="图片5.png" data-nodeid="67509"></p>
<p data-nodeid="67393">综上所述，Zeep 对最新版本的 Python 支持的更好，而且没有性能问题。如果你的项目是新设立的，在选用Web Service客户端时，不妨直接使用Zeep。</p>
<h3 data-nodeid="67394">总结</h3>
<p data-nodeid="67395">本章节中我们重点介绍了 Web Service 及 Web Service 接口，尤其是以 WSDL 格式提供的接口应该如何测试。 并且介绍了两个测试Web Service的Python 库suds和Zeep， 以及在日常工作中，我们是如何使用suds或者Zeep来封装Web Services接口的。</p>
<p data-nodeid="67396">因为很难找到免费、复杂的 Web Service 接口供调用并演示，故本节内容中，代码比较简单。在此布置一个课后作业给你：请你询问下自己公司的开发，请他提供给你一个基于你公司业务的 WSDL 接口，并且根据今天所讲的内容，采用 suds 或者 Zeep 库来执行 Web Service 接口测试，巩固所学。</p>
<p data-nodeid="69519" class="">好的，我是蔡超，我们下节课再见。</p>







<p data-nodeid="70634">在我的公众号 iTesting 中，也有关于 Web Service 接口调用的实例，其中包括对 service 和 factory 的调用。你可以关注 iTesting 并回复 WebService 查看。</p>
<hr data-nodeid="71089">
<p data-nodeid="71090" class="te-preview-highlight"><a href="https://wj.qq.com/s2/7506053/9b01" data-nodeid="71093">课程评价入口，挑选 5 名小伙伴赠送小礼品～</a></p>

---

### 精选评论


