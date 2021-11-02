<p style="text-align: justify; line-height: 1.75em;"></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本课时我们进入 Page Object 模式的讲解，首先我们来了解 Page Object 模式的基础概念。</span></p>
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">传统测试用例问题</span></p></h2>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/6F/11/Cgq2xl5gs92AIuvBAAP8QZgz0qs996.png"></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们先来看下传统的测试用例，这是企业微信添加成员的一个测试用例，我们可以看到测试用例里开始使用 setup 初始化 self.driver，初始化之后通过 self.driver.get 打开一个网址，然后使用显式等待等待一个对象出现，然后通过 find_element 完成一系列的输入，通过 click 完成点击，最后进行断言，这是一个传统的测试用例的经典流程。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">但基于传统的测试用例会存在以下问题：</span></p>
<ul>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">无法适应 UI 变化，UI 变化会导致大量的 case 需要修改；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">大量的样板代码 driver find click；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">一旦出现非预期的弹框阻挡就会失败；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">无法清晰表达用例场景。</span></p></li>
</ul>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">为了解决以上这些问题，行业里就演变出了 Page Object Model（POM）模式，简称为 PO，PO的发展历史有几个关键的时间节点，首先是 2013 年，测试研发领域的大神 Martin Flower 发表了一篇专门介绍 PO 的文章，到了 2015 年，Selenium 官方正式引入 PO 模式。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">PO 模式的普及就涉及一位大神马丁<span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51); font-size: 12pt;">·福勒，目前是 ThoughtWorks &nbsp;的 CTO，他在研发和测试领域都有非常多的著作，对设计模式也有非常深入的理解。</span></span></p>
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">PO 封装</span></p></h2>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51); font-size: 12pt;">在 2013 年的时候，</span>马丁<span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51); font-size: 12pt;">·福勒写了一篇文章，这篇文章提到当你针对 Web 页面编写一个测试用例的时候，就需要引用里面的一些控件，去完成对应的操作与判断，但是如果你编写一个测试用例去操作 HTML，你的测试用例面对 UI 的变化时就会变得非常脆弱。如果用一个 Page Object 封装了一个 HTML 页面，或一个片段，通过一个面向应用的 &nbsp;API，就可以允许你操作底层页面时不再需要关注底层页面的细节。</span></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/6F/10/CgpOIF5gs92AGCGXAAGM0fHgeW4741.png"></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这里举个示例，你可以看到示例中首先有一个面向应用的 API，这个 API 完成关键业务的描述，比如完成具体的业务，然后下面有一条 API 是关于 H5 操作的，也就是拆分了一层逻辑。我们通过抽象一个面向应用的 API，就可以让关于 H5 自动化的逻辑隐藏到底层，当页面发生变化的时，只需要修改底层的一个定义，上层的 case 则没有任何影响。</span></p>
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">六 大原则</span></p></h2>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">那么封装的逻辑首先以页面为单位去建模，然后隐藏内部的实现细节，本质上是一个面向抽象的编程，优点是大量的 find、click 在测试用例里看不到了，提高了测试用例的易读性，因为修改对测试用例的影响非常小，通过 PO 封装就可以实现修改范围可控，只需要在底层修改一个地方，就可以保证原有测试用例本身没有什么变化，因为修改也是在 PO 层面进行的，测试用例基本是保持不变，这就是 PO 模式的一个优点。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/6F/11/Cgq2xl5gs92ABdGpAAGvfSpvxmM165.png"></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Selenium 把 PO 引入到官方文档后总结出了 6 条原则。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"> </span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我用中文来概括这 6 大原则，主要可以分为两个方向，一个是方法意义，一个是字段意义。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">类中的方法代表的意义：</span></p>
<ul>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">用公共方法代表 UI 所提供的功能，比如页面提供的功能分别用方法来对应；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">方法应该返回其他的 PO 或者返回用于断言的数据，也可以用来返回自身；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">同样的行为不同的结果可以建模为不同的方法，比如说登录成功或失败的场景，登录成功会跳转到首页，登录失败仍然会停留在当前页，类似于这种场景的返回值是不一样的，它认为你可以建模成不同的方法；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">不要在方法内加断言。</span></p></li>
</ul>
<p style="margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; margin-top: 5px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">PO中的字段代表的意义：</span></p>
<ul>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">不要暴露页面内部的元素给外部，比如你有几个控件不要让测试用例知道，你只需要在 PO 内定义一个私有变量就可以了；</span></p></li>
 <li><p style="line-height: 1.75em; margin-bottom: 5px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">不需要建模 UI 内的所有元素，一个页面有非常多的控件你也不需要将所有控件都表示出来，用到那个就写那个。</span></p></li>
</ul>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">比如说登录场景，假如有一个登录页面提供了两个功能，一个是登录功能，一个是找回密码，然后我们就可以创建一个 PO，这个 PO 叫作 Login 类，里面分别对应创建登录和找回密码2个方法，这也是最直接的封装的思路，在登录页面内有多少元素我们是不需要关心的，把细节隐藏在 PO 的内部，不要让它们污染到测试用例。mm</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">同时登录有成功、失败两个场景，我们需要把登录方法再次拆开，拆成一个成功的登录和失败的登录，成功的登录会进入首页它应该返回首页的 PO，而登录失败它需要返回当前页面，有一些重要的内容我们是需要断言的，我们可以提供一些方法让断言的内容返回，在测试用例中用断言判断是否符合预期。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">上面是关于 PO 怎么抽象方法的示例，接下来我们看下 Java、Python 给我们提供了什么样的封装方法。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在 Java 客户端里提供了 page factory + @FindBy 注解的封装方式，以方便你构建自己的 PO。Python 里目前没有很成熟的方式，只有最近 Mozilla 公司开源的 PyPOM，当然这个也封装的比较基础。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">总体而言，各个语言的 binding 都实现了对 PO 的简单封装，但是这些封装太简单，在我们实际工作中一般不能满足我们的需要，所以不少公司都会在这些封装的基础上实现自己对应的封装，所以到目前为止，各家公司都很少使用各个语言默认的封装方法，而是自己定制实现对 PO 的封装。</span></p>
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">基于 PO 用例组织结构</span></p></h2>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如果基于 PO 思想进行封装，我们的用例通常分为以下几种对应的目录：</span></p>
<ul>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">创建 page 包，完成对页面的封装；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">创建 testcase 包，调用各类 page 完成业务流程并进行断言，所以 testcase主要用来存储测试用例，这里面的测试用例没有任何关于 H5 的操作，它只有面向业务应用的 API；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">data：通常用来存放配置文件和数据驱动；</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">utils：如果你的测试用例需要更多便捷的功能封装，通常在 utils下进行。</span></p></li>
</ul>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">整体上 page 和 testcase 是比较重要的两个目录。</span></p>
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">编写用例顺序</span></p></h2>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">基于 PO 模式我们怎么去编写一个测试用例呢，首先我们需要了解页面的功能，比如上面提到的登录页面，涉及用户名和密码，我们就需要针对这个页面创建一个 PO 类，然后分别创建对应需要的方法，比如说登录成功和失败、忘记密码。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在编写测试用例时，通常方法体内我们是不用着急写具体实现的，而是在方法体的定义里先直接写一个 pass，然后开始编写测试用例，并把 PO 的各个功能连接起来，在测试用例里调用 PO 的方法来完成入参、返回值，以及断言等具体数据的定义。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">一旦编写完用例的断言之后，我们就可以重新实现 PO 内的方法了，将原有 pass 内容改写成与自动化框架结合的具体实现，实现完成之后就可以运行了，测试通过就代表整个 case 没有问题，整个实现风格类似于 TDD 风格，也就是测试驱动开发的风格编写 PO 用例。</span></p>
<p></p>

---

### 精选评论


