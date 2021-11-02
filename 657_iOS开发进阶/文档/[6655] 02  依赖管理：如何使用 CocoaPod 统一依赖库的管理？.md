<p data-nodeid="1721" class="">在 iOS App 开发方面，几乎所有的 App 都需要使用到第三方依赖库。依赖库不仅能为我们提供丰富的功能，还能避免我们从头开发，在节省时间的同时也减少许多 Bug 。</p>
<p data-nodeid="1722">但伴随着软件功能越来越丰富，依赖库数量越来越多，由此也出现了“依赖地狱”，比如依赖库循环依赖，底层依赖库版本冲突等。为了解决此类问题，于是，依赖库管理工具也就出现了。</p>
<p data-nodeid="1723">目前流行的依赖库管理工具主要有：Git Submodules、Carthage、 Swift Package Manager 和 CocoaPods。在这里我们选择 CocoaPods。为什么呢？原因有三：</p>
<ol data-nodeid="1724">
<li data-nodeid="1725">
<p data-nodeid="1726">CocoaPods 非常成熟，十分稳定，并且简单易用，学习成本低，效果明显；</p>
</li>
<li data-nodeid="1727">
<p data-nodeid="1728">CocoaPods 会自动整合 Xcode 项目，使得其他项目成员在使用第三方库时无须任何额外的手工操作；</p>
</li>
<li data-nodeid="1729">
<p data-nodeid="1730">CocoaPods 已经成为 iOS 业界标准，支持几乎所有的开源库和商业库，即便是 Objective-C 的依赖库以及二进制文件（binary）依赖库，CocoaPods 也提供支持。</p>
</li>
</ol>
<p data-nodeid="1731">那么，怎样使用 CocoaPods 来管理第三方依赖库呢？接下来我会从语义化版本管理、Pod 版本管理、Pod 版本更新三个方面展开介绍。</p>
<h3 data-nodeid="1732">语义化版本管理</h3>
<p data-nodeid="1733">开发软件，免不了要更新迭代，所以每一次更新的版本号管理变得很重要。并且，一旦版本号混乱，就会导致一系列问题，比如很难查找和修改线上崩溃，没办法支持多团队并行开发，等等。为了避免此类问题，我们可以使用语义化版本管理（Semantic Versioning）来统一版本号的定义规范。</p>
<p data-nodeid="1734">语义化版本号是一种通用的版本号格式规范，目前绝大部分优秀的第三方依赖库都遵循这一规范来发布版本。</p>
<p data-nodeid="1735">具体来说，语义化版本号的版本号一般包括四部分：MAJOR、MINOR、PATCH、BUILD。每一部分都由递增的数值组成，例如 1.2.3.4，其中 1 是MAJOR， 2 是 MINOR。如果我们更新 MINOR 版本号，那么下一个版本就是 1.3.0.0。接下来我详细介绍下这四部分。</p>
<ul data-nodeid="1736">
<li data-nodeid="1737">
<p data-nodeid="1738">MAJOR 是指主版本号，通常在重大更新的时候才会需要更新主版本号。例如 iOS 每年都会更新一个主版本号。而对于第三方库来说，主版本号的更新，表示该库的 API 新增了重大功能，或者引入了不可兼容的更新 （breaking changes）。</p>
</li>
<li data-nodeid="1739">
<p data-nodeid="1740">MINOR 是指副版本号，用于小功能的改善。例如 iOS 14 在发布主版本后，在一年内可能发布多个副版本如 14.1、 14.2 来完善其系统功能。一般对于第三方库来说，副版本的更新就是新增一些 API，但不包含不可兼容的更新。</p>
</li>
<li data-nodeid="1741">
<p data-nodeid="1742">PATCH 是指补丁版本号，一般用于 bug fix 以及修复安全性问题等。对于第三方库来说，补丁版本号的更新也不应该有不可兼容的更新。虽然实际操作中这会有些困难，但我们可以通过把原有 API 标记为 deprecated，或者为新 API 参数提供默认值等办法来解决。</p>
</li>
<li data-nodeid="1743">
<p data-nodeid="1744">BUILD 是指构建版本号，通常在内部测试时使用。一般当我们使用 CI 服务器进行自动构建时，构建版本号会自动更新。</p>
</li>
</ul>
<h3 data-nodeid="1745">Pod 版本管理</h3>
<p data-nodeid="1746">要使用 CocoaPods 管理第三方依赖库，首先要新建一个 Podfile 文件，然后执行<code data-backticks="1" data-nodeid="1868">bundle exec pod install</code>命令来安装所有依赖库。这时候 CocoaPods 会自动帮我们建立一个 Podfile.lock 文件和一个 Workspace文档。</p>
<p data-nodeid="1747">注意，在第一讲我们说过，由于是通过 Bundler 来安装 CocoaPods，每次执行<code data-backticks="1" data-nodeid="1871">pod</code>命令前，都需要加上<code data-backticks="1" data-nodeid="1873">bundle exec</code>。不过为了简洁，后面涉及<code data-backticks="1" data-nodeid="1875">pod</code>命令时，我会省略<code data-backticks="1" data-nodeid="1877">bundle exec</code>部分。</p>
<p data-nodeid="1748">接下来，我详细介绍下 Podfile 文件、 Podfile.lock 和 Workspace 文档到底是什么，以及如何使用。</p>
<h4 data-nodeid="1749">Podfile 文件</h4>
<p data-nodeid="1750">Podfile 文件是一个配置文件，它主要是用来描述 Xcode 项目里各个 target 的依赖库。我们项目的 Podfile 文件可以在<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Podfile" data-nodeid="1884">仓库中</a>找到。在这里，我主要和你介绍一下 Podfile 文件中的几个重要配置。</p>
<p data-nodeid="1751"><strong data-nodeid="1889">source 配置</strong></p>
<p data-nodeid="1752"><code data-backticks="1" data-nodeid="1890">source</code>用于指向 PodSpec（Pod 规范）文件的 Repo，从而使得 CocoaPods 能查询到相应的 PodSpec 文件。</p>
<p data-nodeid="1753">具体来说，当使用公共依赖库的时候，<code data-backticks="1" data-nodeid="1893">source</code>需要指向 CocoaPods Master Repo，这个主仓库集中存放所有公共依赖库的 PodSpec 文件。 由于 CocoaPods 经常被开发者吐槽 Pod 下载很慢，因此 CocoaPods 使用了 CDN （Content Delivery Network，内容分发网络）来缓存整个 CocoaPods Master Repo， 方便开发者快速下载。具体的配置方法就是使<code data-backticks="1" data-nodeid="1895">source</code>指向 CND 的地址，代码示例如下：</p>
<pre class="lang-java" data-nodeid="1754"><code data-language="java">source <span class="hljs-string">'https://cdn.cocoapods.org/'</span>
</code></pre>
<p data-nodeid="1755">如果使用的是私有依赖库，我们也需要把<code data-backticks="1" data-nodeid="1898">source</code>指向私有库的 PodSpec Repo，以使得 CocoaPods 能找到相应的 PodSpec 文件。 代码示例如下：</p>
<pre class="lang-java" data-nodeid="1756"><code data-language="java">source <span class="hljs-string">'https://my-git-server.com/internal-podspecs'</span>
</code></pre>
<p data-nodeid="1757">注意，当我们使用私有库时，执行<code data-backticks="1" data-nodeid="1901">pod install</code>命令的机器必须能访问到<code data-backticks="1" data-nodeid="1903">source</code>所指向的 Repo。</p>
<p data-nodeid="1758"><strong data-nodeid="1908">project 和 workspace</strong></p>
<p data-nodeid="1759"><code data-backticks="1" data-nodeid="1909">project</code>用于指定我们的主项目文档。该项目文档会使用到 CocoaPods 管理的所有第三方依赖库。</p>
<p data-nodeid="1760"><code data-backticks="1" data-nodeid="1911">workspace</code>用于指定要生成和更新的 Workspace 文档。和其他依赖库管理工具不一样，CocoaPods 会自动生成一个 Workspace 文档，然后我们只能使用该文档而不是 Xcode 项目文档来进行后续开发。</p>
<p data-nodeid="1761">代码示例如下：</p>
<pre class="lang-java" data-nodeid="1762"><code data-language="java">project <span class="hljs-string">'./Moments/Moments.xcodeproj'</span>
workspace <span class="hljs-string">'./Moments.xcworkspace'</span>
</code></pre>
<p data-nodeid="1763">这其中 Moments.xcodeproj 就是我们的主项目文档，它一般放在和项目名字相同的下一层目录下。</p>
<p data-nodeid="1764">而 Moments.xcworkspace 是 CocoaPods 为我们生成的 Workspace文档，为了统一，我建议名字也是和主项目相同。</p>
<p data-nodeid="1765"><strong data-nodeid="1921">platform 和 use_frameworks</strong></p>
<p data-nodeid="1766">先看示例，它表示什么呢？</p>
<pre class="lang-java" data-nodeid="1767"><code data-language="java">platform :ios, <span class="hljs-string">'14.0'</span>
use_frameworks!
</code></pre>
<p data-nodeid="1768">为了保证所有依赖库与主项目在编译和运行时兼容，我们指定的系统版本号需要和主项目所支持的系统版本号保持一致。而<code data-backticks="1" data-nodeid="1924">platform</code>就是用于指定操作系统以及所支持系统的最低版本号。比如，例子中的<code data-backticks="1" data-nodeid="1926">platform :ios, '14.0'</code>就表示支持 iOS 14.0 以上的所有 iOS 版本。</p>
<p data-nodeid="1769">另外一行的<code data-backticks="1" data-nodeid="1929">use_frameworks!</code>这一配置会让 CocoaPods 把所有第三方依赖库打包生成一个动态加载库，而不是静态库。因为动态库是我们经常用到的，它能有效地加快编译和链接的速度。</p>
<p data-nodeid="1770"><strong data-nodeid="1934">组织同类型的第三方依赖库</strong></p>
<pre class="lang-java" data-nodeid="1771"><code data-language="java">def dev_pods
  pod <span class="hljs-string">'SwiftLint'</span>, <span class="hljs-string">'0.40.3'</span>, configurations: [<span class="hljs-string">'Debug'</span>]
  pod <span class="hljs-string">'SwiftGen'</span>, <span class="hljs-string">'6.4.0'</span>, configurations: [<span class="hljs-string">'Debug'</span>]
end
</code></pre>
<p data-nodeid="1772">其中<code data-backticks="1" data-nodeid="1936">configurations: ['Debug']</code>用于指定该依赖库只是使用到<code data-backticks="1" data-nodeid="1938">Debug</code>构建目标（target）里面，而不在其他（如<code data-backticks="1" data-nodeid="1940">Release</code>）构建目标里面，这样做能有效减少 App Store 发布版本的体积。</p>
<p data-nodeid="1773"><code data-backticks="1" data-nodeid="1942">def dev_pods end</code>代码块是“复用同一类依赖库方式”的意思，我们可以把同类型的依赖库都放进这个代码块里面。比如，我们的 Moments 项目中就分别有<code data-backticks="1" data-nodeid="1944">dev_pods</code>（开发相关的库）,<code data-backticks="1" data-nodeid="1946">core_pods</code>（核心库）以及<code data-backticks="1" data-nodeid="1948">thirdparty_pods</code>(第三方库)等代码块定义。</p>
<p data-nodeid="1774"><strong data-nodeid="1953">target 配置</strong></p>
<p data-nodeid="1775">有了这些复用库定义以后，怎样使用到项目的构建目标（target）里面呢？下面就是一个例子。</p>
<pre class="lang-java" data-nodeid="1776"><code data-language="java">target 'Moments' do
  dev_pods
  core_pods
  # other pods...
end
</code></pre>
<p data-nodeid="1777">我们可以把构建目标所使用的所有依赖库放进<code data-backticks="1" data-nodeid="1956">target</code>代码块中间，上面中的<code data-backticks="1" data-nodeid="1958">Moments</code>就是我们的 App 构建目标。该构建目标依赖了<code data-backticks="1" data-nodeid="1960">dev_pods</code>和<code data-backticks="1" data-nodeid="1962">core_pods</code>等各组依赖库。执行<code data-backticks="1" data-nodeid="1964">pod install</code>的时候，CocoaPods 会把<code data-backticks="1" data-nodeid="1966">dev_pods</code>代码块自动展开为<code data-backticks="1" data-nodeid="1968">SwiftLint</code>和<code data-backticks="1" data-nodeid="1970">SwiftGen</code>，那么<code data-backticks="1" data-nodeid="1972">Moments</code>构建目标能使用<code data-backticks="1" data-nodeid="1974">SwiftLint</code>和<code data-backticks="1" data-nodeid="1976">SwiftGen</code>依赖库了。</p>
<p data-nodeid="1778"><strong data-nodeid="1981">依赖库的版本</strong></p>
<pre class="lang-java" data-nodeid="1779"><code data-language="java"> pod <span class="hljs-string">'RxSwift'</span>, <span class="hljs-string">'= 5.1.1'</span>
 pod <span class="hljs-string">'RxRelay'</span>, <span class="hljs-string">'= 5.1.1'</span>
</code></pre>
<p data-nodeid="1780">在 CocoaPods 里面，每一个依赖库称为一个 Pod （注意这里首字母大写，Pod 指一个库），指定一个 Pod 的命令是<code data-backticks="1" data-nodeid="1983">pod</code>（注意这里是小写，表示一条命令）。在 Podfile 里面我们可以通过这样的格式<code data-backticks="1" data-nodeid="1985">pod 'RxSwift', '= 5.1.1'</code>来配置依赖库的版本号。其中，<code data-backticks="1" data-nodeid="1987">RxSwift</code>或者<code data-backticks="1" data-nodeid="1989">RxRelay</code>是依赖库的名字，<code data-backticks="1" data-nodeid="1991">5.1.1</code>为版本号。这些库的名字以及版本号都可以在 CocoaPods 官网上找到。</p>
<p data-nodeid="1781"><strong data-nodeid="1998">为了统一管理第三方依赖库的版本，我建议统一使用 = 来锁定该依赖库的版本，这样就能保证每次执行</strong><code data-backticks="1" data-nodeid="1996">pod install</code>的时候都可以为同一个库下载同一个版本。</p>
<p data-nodeid="1782">除了 = 操作符以外，CocoaPods 还支持其他操作符来指定版本：</p>
<ul data-nodeid="1783">
<li data-nodeid="1784">
<p data-nodeid="1785"><code data-backticks="1" data-nodeid="2000">&gt; 0.1</code>表示大于 0.1 的任何版本，这样可以包含 0.2 或者 1.0；</p>
</li>
<li data-nodeid="1786">
<p data-nodeid="1787"><code data-backticks="1" data-nodeid="2002">&gt;= 0.1</code>表示大于或等于 0.1 的任何版本；</p>
</li>
<li data-nodeid="1788">
<p data-nodeid="1789"><code data-backticks="1" data-nodeid="2004">&lt; 0.1</code>表示少于 0.1 的任何版本；</p>
</li>
<li data-nodeid="1790">
<p data-nodeid="1791"><code data-backticks="1" data-nodeid="2006">&lt;= 0.1</code>表示少于或等于 0.1 的任何版本；</p>
</li>
<li data-nodeid="1792">
<p data-nodeid="1793"><code data-backticks="1" data-nodeid="2008">~&gt; 0.1.2</code>表示大于 0.1.2 而且最高支持 0.1.* 的版本，但不包含 0.2 版本。</p>
</li>
</ul>
<p data-nodeid="1794">这几个操作符相里面，<code data-backticks="1" data-nodeid="2013">~&gt;</code>（Squiggy arrow）操作符更为常用，它是以最后一个部分的版本号（例子中 0.1.2 的最后一个部分是补丁版本号 <em data-nodeid="2019">.</em>.2）来计算可以支持的最高版本号。</p>
<p data-nodeid="1795">例如<code data-backticks="1" data-nodeid="2021">~&gt; 0.1.2</code>表示 &gt;= 0.1.2&nbsp;并且 &lt; 0.2.0，但不能等于 0.2.0， 因为 0.2.0 已经更新了副版本号而不仅仅是补丁版本号了。另外一个例子是<code data-backticks="1" data-nodeid="2025">~&gt; 0.1</code>，表示 &nbsp;&gt;= 0.1&nbsp; 并且 &lt; 1.0，举例来说，我们可以更新到 0.9 但不能更新到 1.0。</p>
<p data-nodeid="1796">前面我介绍的是引用外部的第三方依赖库，如果我们的项目有自己的内部依赖库，要怎样在 CocoaPods 引用它呢？其实很简单，我们可以执行以下命令：</p>
<pre class="lang-java" data-nodeid="1797"><code data-language="java">pod <span class="hljs-string">'DesignKit'</span>, :path =&gt; <span class="hljs-string">'./Frameworks/DesignKit'</span>, :inhibit_warnings =&gt; <span class="hljs-keyword">false</span>
</code></pre>
<p data-nodeid="1798">和其他外部依赖库不一样，我们需要使用<code data-backticks="1" data-nodeid="2031">:path</code>来指定该内部库的路径。</p>
<h4 data-nodeid="1799">Podfile.lock 文件</h4>
<p data-nodeid="1800">Podfile.lock 文件是由 CocoaPods 自动生成和更新的，该文件会详细列举所有依赖库具体的版本号。比如，</p>
<pre class="lang-java" data-nodeid="1801"><code data-language="java">DEPENDENCIES:
&nbsp; - Alamofire (= <span class="hljs-number">5.2</span><span class="hljs-number">.0</span>)
&nbsp; - Firebase/Analytics (= <span class="hljs-number">7.0</span><span class="hljs-number">.0</span>)
PODFILE CHECKSUM: <span class="hljs-number">400</span>d19dbc4f5050f438797c5c6459ca0ef74a777
</code></pre>
<p data-nodeid="1802">当执行<code data-backticks="1" data-nodeid="2036">pod install</code>后，CocoaPods 会根据 Podfile 文件解释出各依赖库的特定版本号，然后一一列举在 DEPENDENCIES 下面。在上述的例子中，我们的 App 在构建过程中使用了5.2.0 的 Alamofire 库以及 7.0.0 的 Firebase Analytics 库。</p>
<p data-nodeid="1803">PODFILE CHECKSUM 用于记录 Podfile 的验证码，任何库的版本号的更改，都会改变该验证码。这样能帮助我们在不同的机器上，快速检测依赖库的版本号是否一致。</p>
<p data-nodeid="1804">我建议要把 Podfile 和 Podfile.lock 文件一同 commit 并 push 到 Git 代码管理服务器里面。特别是在团队开发的环境下，这样能帮助我们保证各个依赖库版本号的一致性。</p>
<p data-nodeid="1805">在实践操作中，无论我们在哪台机器上执行<code data-backticks="1" data-nodeid="2041">pod install</code>， PODFILE CHECKSUM 都不应该发生任何改变。因为我们在 Git 保存了 Podfile.lock，一旦我们发现老版本 App 的 Bug ，就可以根据该文件为各个依赖库重新安装同一版本号，来重现和定位问题，从而帮助我们快速修改这些 Bug。</p>
<h4 data-nodeid="1806">Workspace 文档</h4>
<p data-nodeid="1807">Workspace 文档是 Xcode 管理子项目的方式。通过 Workspace，我们可以把相关联的多个 Xcode 子项目组合起来方便开发。</p>
<p data-nodeid="1808">前面说过，当我们执行<code data-backticks="1" data-nodeid="2046">pod install</code>的时候，CocoaPods 会自动创建或者更新一个叫作 Pods 的项目文档（Pods.xcodeproj&nbsp;）以及一个 Workspace 文档（在我们项目中叫作 Moments.xcworkspace）。</p>
<p data-nodeid="1809">其中，Pods 项目文档负责统一管理各个依赖库，当我们在 Podfile 里面指定构建成动态库的时候，该项目会自动生成一个名叫<code data-backticks="1" data-nodeid="2049">Pods_&lt;项目名称&gt;.framework</code>的动态库供我们项目使用。</p>
<p data-nodeid="1810">而 Workspace 文档则统一管理了我们原有的主项目 （Moments.xcodeproj）以及那个 Pods 项目。</p>
<p data-nodeid="1811">与此同时，CocoaPods 还会修改 Xcode 项目中的 Build Phases&nbsp; 以此来检测 Podfile.lock 和 Manifest.lock 文件的一致性，并把<code data-backticks="1" data-nodeid="2053">Pods_&lt;项目名称&gt;.framework</code>动态库嵌入我们的主项目中去。</p>
<p data-nodeid="1812">以上所有操作都是由 CocoaPods 自动帮我们完成。以后的开发，我们都可以打开 Workspace 文档而不是原有的 Xcode 项目文档来进行。</p>
<h3 data-nodeid="1813">Pod 版本更新</h3>
<p data-nodeid="1814">使用 CocoaPods 管理第三方依赖库的操作非常简单，可是一旦使用不当，特别是在 Pod 更新的时候，很容易引起依赖库版本不一致，从而出现各种问题。</p>
<p data-nodeid="1815">比如，在编译程序的时候，有些开发者可以顺利进行，而另外一些开发者编译时候就会出错；或者程序在本地编译时运行良好，一旦在 CI 上构建，就会出现 App 崩溃，等等。</p>
<p data-nodeid="1816">那么，怎么保证更新 Pod 的时候都能保证版本一致呢？</p>
<p data-nodeid="1817">下面结合我的实践经验，以第三方网络库 Alamofire 为例子和你介绍下。</p>
<p data-nodeid="1818">第一步，CocoaPods 已经为我们提供了<code data-backticks="1" data-nodeid="2062">pod outdated</code>命令，我们可以用它一次查看所有 Pod 的最新版本，而无须到 GitHub 上逐一寻找。下面是执行<code data-backticks="1" data-nodeid="2064">pod outdated</code>命令的其中一条结果：</p>
<pre class="lang-java" data-nodeid="1819"><code data-language="java">The following pod updates are available:
- Alamofire <span class="hljs-number">5.2</span><span class="hljs-number">.0</span> -&gt; <span class="hljs-number">5.2</span><span class="hljs-number">.0</span> (latest version <span class="hljs-number">5.4</span><span class="hljs-number">.0</span>)
</code></pre>
<p data-nodeid="1820">这表示当前我们使用了版本为 5.2.0 的 Alamofire ，其最新版本为 5.4.0。如果我们决定更新到版本 5.4.0，那么可以继续下一步。</p>
<p data-nodeid="1821">第二步，在更新依赖库版本之前，为了避免在新版本中不小心引入 Bug，我们需要了解新的版本到底提供了哪些新功能，修改了哪些 Bug，与老版本是否兼容等事项。具体我们可以到 CocoaPods 官网上查找需要更新的第三方依赖库，然后在 GitHub 等平台上找到，并仔细阅读该库的版本说明（release note）。</p>
<p data-nodeid="1822"><strong data-nodeid="2072">请注意，我们要阅读当前使用版本到要更新的版本之间的所有版本说明。</strong> 在这个例子中，我们要阅读 5.2.1，5.2.2，5.3.0 和 5.4.0 的所有版本说明。这些版本说明会列出新增功能，更新的 API，修改的 Bug，有没有不可兼容的更新 。</p>
<p data-nodeid="1823">第三步，在 Podfile 文件里把要更新的 Pod 的版本号进行修改。例如把<code data-backticks="1" data-nodeid="2074">pod 'Alamofire', '= 5.2.0'</code>改成<code data-backticks="1" data-nodeid="2076">pod 'Alamofire', '= 5.4.0'</code>。 然后执行<code data-backticks="1" data-nodeid="2078">pod install</code>来重新生成 Podfile.lock 文件。</p>
<p data-nodeid="3014" class="te-preview-highlight">此时特别注意的是，我们要使用<code data-backticks="1" data-nodeid="3016">pod install</code>而不是<code data-backticks="1" data-nodeid="3018">pod update</code>。因为执行<code data-backticks="1" data-nodeid="3020">pod update</code>会自动更新所有 Pod 的版本，这可能会更新了一些我们目前还不想更新的 Pod，从而会引入一些难以觉察的问题。</p>


<p data-nodeid="1825">第四步，如果所更新的版本包含了不可兼容的更新，我们需要修改代码来保证代码能顺利完成编译。</p>
<p data-nodeid="1826">第五步，很多第三方依赖库都是一些通用的基础组件，一旦发生问题会影响到整个 App 的功能，因此我们需要根据所更新的库进行回归测试。例如当更新了 Alamofire 库的时候，我们需要把每个网络请求都执行一遍，避免所更新的版本引入新的 Bug。</p>
<p data-nodeid="1827">第六步，为了把更新的版本共享给所有开发者和 CI 服务器，我们需要把 Podfile 和 Podfile.lock 文件一同 commit 并 push 到 Git 代码管理服务器，并通过 Pull Request 流程并入主分支。</p>
<p data-nodeid="1828">第七步，一旦更新的代码并入主分支后，要通过 Slack 等内部通信软件告诉所有开发者 pull 或者 rebase 主分支的代码，并执行<code data-backticks="1" data-nodeid="2095">pod install</code>来更新他们开发环境的所有依赖库。</p>
<p data-nodeid="1829"><strong data-nodeid="2104">特别注意，千万不要使用</strong><code data-backticks="1" data-nodeid="2100">pod update</code>，因为<code data-backticks="1" data-nodeid="2102">pod update</code>会自动把开发者机器上所有 Pod 的版本自动更新了。这种更新往往不是我们想要的结果，我们希望统一更新各个 Pod 的版本，并通过 Git 进行集中管理。</p>
<p data-nodeid="1830">如果开发者在编译新代码前没有执行<code data-backticks="1" data-nodeid="2106">pod install</code>命令，会出现以下的错误。</p>
<blockquote data-nodeid="1831">
<p data-nodeid="1832">The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation.</p>
</blockquote>
<p data-nodeid="1833">这错误可以有效提醒所有开发者，需要再次执行<code data-backticks="1" data-nodeid="2114">pod install</code>来更新他们本地的依赖库，从而保证所有开发者使用的依赖库的版本都是一致的。</p>
<p data-nodeid="1834">另外，如果更新了基础组件的依赖库（如网络库），在测试阶段，我们还需要进行全面的回归测试。因为这些基础组件库的新版本如果有 Bug 很可能导致我们的 App 会发生大比例的崩溃，严重影响用户的体验。</p>
<p data-nodeid="1835">有了上面的一流程，我们就可以有效地保证每个开发者使用的依赖库版本都是一致的，同时也能保证 CI 在自动构建 App 的时候所使用的依赖库版本也是统一的。</p>
<h3 data-nodeid="1836">总结</h3>
<p data-nodeid="1837">这一讲我介绍了如何使用 CocoaPods 来统一管理依赖库的版本。特别是根据我自己的经验总结了一套更新 Pod 版本的流程，希望你灵活使用这些步骤，从而少走弯路。</p>
<p data-nodeid="1838"><img src="https://s0.lgstatic.com/i/image6/M00/0A/3A/CioPOWA3GR6AIe4nAApbaq5B2h0899.png" alt="依赖管理.png" data-nodeid="2122"></p>
<p data-nodeid="1839">这里我再特别强调一下，为了保证依赖库版本都能保持一致，尽量不要执行<code data-backticks="1" data-nodeid="2124">pod update</code>，而是使用通过修改 Podfile 文件里的版本号并执行<code data-backticks="1" data-nodeid="2126">pod install</code>来更新 Pod 的版本，然后把 Podfile 和 Podfile.lock 文件一同并入 Git 主分支中进行统一管理。</p>
<p data-nodeid="1840">思考题：</p>
<blockquote data-nodeid="1841">
<p data-nodeid="1842">CocoaPods 非常简单易用，它可以同时管理依赖库的依赖项，例如我们的 App 依赖 A 库， 而 A 库又依赖 B 库，同时 B 库依赖 C 库，CocoaPods 可以帮我们自动找出所有依赖项并按顺序安装所有依赖库。 那你知道 CocoaPods 是如何管理依赖库的依赖呢？</p>
</blockquote>
<p data-nodeid="1843">下一讲，我将为你介绍如何统一构建配置。</p>
<p data-nodeid="1844">源码地址：</p>
<blockquote data-nodeid="1845">
<p data-nodeid="1846">Podfile 文件地址：<br>
<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Podfile" data-nodeid="2136">https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Podfile</a></p>
</blockquote>
<hr data-nodeid="1847">
<p data-nodeid="1848"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="2141"><img src="https://s0.lgstatic.com/i/image6/M00/08/77/Cgp9HWA0wqWAI70NAAdqMM6w3z0673.png" alt="Drawing 1.png" data-nodeid="2140"></a></p>
<p data-nodeid="1849"><strong data-nodeid="2145">《大前端高薪训练营》</strong></p>
<p data-nodeid="1850" class="">12 个月打磨，6 个月训练，优秀学员大厂内推，<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="2149">点击报名，高薪有你</a>！</p>

---

### 精选评论

##### *虎：
> 如何回复讲师的回复？只能创建新的留言回复吗？

 ###### &nbsp;&nbsp;&nbsp; 官方客服回复：
> &nbsp;&nbsp;&nbsp; 是的，想要回复讲师的答复，可以在新留言里面的开头标记说明就行

##### *虎：
> "第七步，一旦更新的代码并入主分支后，要通过 Slack 等内部通信软件告诉所有开发者 pull 或者 rebase 主分支的代码，并执行pod install来更新他们开发环境的所有依赖库"能否把更新的依赖库也一并 push 到远端仓库，这样团队里其他开发人员就不需要额外 pod install 了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 看得很仔细哦，更新和 push 到远程仓库的是 Podfile 以及 Podfile.log 文件，团队其他人员还是需要执行 pod install 安装并更新 Pods 文件夹，这个 Pod 文件夹存放的是所有第三方库的源码文件，我们一般并不把该文件夹放到 Git 里面。因此每个开发环境都需要执行 pod install 来更新。文章中提到，如果开发者不执行 pod install 是没有办法编译项目的，并且有错误提示。

##### **聪：
> 还不如直接看官网来的详细和简单！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哈哈，技术的准确性都以官方文档为准哦。现在 iOS 开发已经不是学习了系统提供的 API 就可以了，还需要学习系统架构，自动化，代码规范，单元测试。在团队协助的时候还需要制定代码管理流程与规范。目前这些内容在苹果官网都没有提供官方的方案，我们的课程是通过一个类似朋友圈 App 作为案例把开发过程中的各个方面连贯起来讲述，希望能成为官方文档的一个补充。

##### **华：
> 老师讲的比较细

##### **不吃窝边草：
> CocoaPods 管理依赖库的依赖，这种A依赖B，B依赖C,不是链表么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 有时候并不是单链条的链表那么简单哦，可能是一个图，要为图找出依赖关系，需要使用拓扑排序。

##### **昆：
> 老师你好，关于pod update的描述，我有个疑问：如果是使用了 podName，'=version' 这时候 通过pod update 也会使用 = 号的版本号，而不会去所有都更新吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是的哦，一旦执行 pod update，会把版本号自动更新了。我们应该手动更新版本后，然后执行 pod install 来安装。

##### *一：
> 感谢老师的讲解😀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 也感谢你的支持。

##### *辰：
> 每个库的PodSpec文件里会配置它所依赖的库，CocoaPods 就是根据这个文件找出库的依赖库，依次类推。

##### *潇：
> 老师，如果pod里面的第三方库都指定了准确的版本号，是不是用 pod update 也可以？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是的，如果我们执行了 pod update 命令，就会把执行的机器中的第三方库版本给更新了，这不是我们想要的效果，为了保证整个团队都使用统一的版本，我们应该让一个人更新版本，并在测试完毕后合并到主分支供大家使用，一个团队在一个时间只有一个人更新第三方库的版本，其他人使用 pod install 来使用就可以了。

##### **超：
> Cocopods 依赖库冲突怎么解决？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请把问题描述清楚哦，这样我好方便回答。

##### **龙：
> 可能我自己写的不太清楚，思考题中，确实还不了解是通过什么算法来管理依赖库的依赖呢，这方面还请赐教。上网查了一些，不过总觉得不对。有关于第三库的管理和使用上，我举个例子，比如我使用Alamofire，一般会在项目中再封装一层，比如叫HttpUtils，而这个HttpUtils我通过Swift Package进行本地化的组件化管理，如果这个HttpUtils写的好，就放在公司内网，大家都可以使用。不知道这样表达是否清晰？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我想问的是 CocoaPods 的内部实现，想让大家不仅会用，还知道里面是怎样实现的，一旦懂了原理，替换成不同的包管理工具都有一致的思路。你说到的情况（HttpUtils）其实是对的，我们在 Moments App 里面也有一个底层网络通信模块，在 APISession 里面。我们也可以把它独立出来成为一个 DesignKit 这样的组件。但是只要逻辑是结耦（目前已经是了），物理上的分离只是水到渠成的事情。同理，路由模块，数据存储那些都可以独立出来。

##### **泽：
> 老师，我自己新建了一个Moments工程， pod install以后，像这些Moments.xcodeproj/project.pbxprojMoments.xcodeproj/project.xcworkspace/contents.xcworkspacedataMoments.xcodeproj/xcuserdata/guoruize_account.xcuserdatad/xcschemes/xcschememanagement.plist这样的文件需要忽略吗， 还是让git管理起来。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 需要呀，请看看我们的  .gitignore 文件，在这里 https://github.com/lagoueduCol/iOS-linyongjian/blob/main/.gitignore#L122

##### **泽：
> CocoaPods是通过.podspec文件 来管理依赖库，在这个文件里配置依赖库的依赖。不知道对不对。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你说得对，第三方自己也使用 .podspec 文件来指明依赖库，可以一层层去找出所有依赖关系。

##### **龙：
> 在.podsepc文件中，有一个dependency参数来管理依赖库的依赖我现在的思路是用cocopods来管理第三方，考虑第三方一般都需要自己再封装一层，在现有项目中再创建SwiftPackage进行封装。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦哦，我不是很明白你说的办法，如果你想回答的是思考题的问题，我想问在 CocoaPods 里面到底是怎样计算出依赖项之间的依赖顺序，我的提示是它使用了一个算法，看看你能不能找出来哦。

##### **钦：
> 如果有一些自己搭建的私有pod ，合适在放在哪一步把私有的pod源引入呢？。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我们会在 第 08 讲｜设计组件：DesignKit 组件桥接设计与开发规范 讲述如何创建和使用私有 Pod。

##### **永：
> 每次思考的问题都没有配答案的吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 后面会有编程题，那些题目比较容易验证，其他的思考题是想让大家多思考并把想法发出来，大家相互启发，共同学习与进步。

##### **泽：
> 老师您好，在团队项目中，像Podfile文件更改，只需要一个人去做吧，比如A把依赖1的版本改了， 执行了pod install命令， 在本地开发没问题，推到了git， B把依赖2的版本改了， 也推到了git，是不是团队成员每个人拉下来代码都需要先进行一次pod install，以保证最新的。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的呀，如果 Git 上的 Podfile 改动了，不执行 pod install 是不能编译项目的，所以你可以先编译，发现有问题再执行  pod install 来重新安装 Pod。

##### **6158：
> 谢谢老师的回复，我的意思是pod 'DesignKit', :path = false这里，在我看来是指向了一个本地路径？如果我的理解正确的话，那么是否所有设备都需要在相同的本地路径里保存一份相关代码？我目前只用过:git =，我的理解是:path =的区别只是一个在本地一个在git服务器上？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，path 指本地路径，可以在代码库里面看到 DesignKit 在 Moments App 下的一个子目录，位于 Frameworks 里面。当我们 Git clone Moments 的 Repo 时就包含了 DesignKit 的源码了。如果有需要可以把 DesignKit 变成一个独立的 Repo，这个在 08 讲会详细讲述。

##### **彬：
> #我提问：Required ruby-2.7.1 is not installed.To install do: 'rvm install "ruby-2.7.1"'只要我在终端cd 到项目文件夹目录下，就会有这样的提示，我明明有装rbenv，有装ruby 2.7.1，那这个问题怎么解决?#讲师回复：请执行一下 rbenv versions 看一下到底有没有安装成功，这个安装过程需要一段时间哦。#回复讲师：我查看了一下 rbenv -v 是能查看到版本">1.1.2 的。但是我看到了，我电脑上是安装了 rvm 的，我现在卸载 rvm 看看，可能是 rvm 和 rbenv 的冲突

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的呀，RVM 会和 rbenv 冲突的呀，我们在文章里面也提到过。如果可以，请只使用 rbenv。

##### **彬：
> post_install do |installer| installer.pods_project.targets.each do |target| target.build_configurations.each do |config| config.build_settings.delete 'IPHONEOS_DEPLOYMENT_TARGET' config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64" end endend；；；Podfile中的这段代码没看懂是什么作用的。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以看一下这个 commit，这段代码用于解决一个构建时候的 warning https://github.com/lagoueduCol/iOS-linyongjian/commit/d30b4048aec9d1bc76626e410c83f208758c54f3#diff-8f7d6adf31268a2d897ee34bd170592648d6e520aa237104395e4a4438af50cb

##### *鹏：
> 您好，使用cocoapods依赖自己的库，是要将自己的库放到本地统一管理起来，然后使用:path链接到本地的路径吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，我们会在第 08 讲讲述如何封装自己的 Pod。使用的时候就正如你说的那样
pod 'DesignKit', :path => './Frameworks/DesignKit', :inhibit_warnings => false

##### **6158：
> 请问内部依赖库是放在本地的吗？如果是的话，是需要所有设备都保存一份本地文件吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问你说是这些依赖库的源代码文件吗？我们只需要把 Podfile 和 Podfile.log 文件保存到 Git 服务器上，每次执行 bundle exec pod install 都能重新下载统一版本的依赖库到 Pods 文件夹里面。Pods 文件夹不需要放到 Git 服务器中。

##### *召：
> 团队成员的cocoapods的版本号也得保持一致

##### **彬：
> Required ruby-2.7.1 is not installed.To install do: 'rvm install "ruby-2.7.1"'只要我在终端cd 到项目文件夹目录下，就会有这样的提示，我明明有装rbenv，有装ruby 2.7.1，那这个问题怎么解决

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请执行一下  rbenv versions 看一下到底有没有安装成功，这个安装过程需要一段时间哦。

##### **9149：
> pod outdated这个好

##### **6126：
> 把source从https://cdn.cocoapods.org/改成https://github.com/CocoaPods/Specs.git后执行pod install提示 如下信息，根据提示修改还是无法解决Unable to add a source with url `https://github.com/CocoaPods/Specs.git` named `cocoapods`.
You can try adding it manually in `/Users/mengmengzhang/.cocoapods/repos` or via `pod repo add`.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哈哈，我也没有遇到个这个问题呀，但是临时 Google 了一下，找到答案啦，比如 Github 里面的回复 https://github.com/CocoaPods/CocoaPods/issues/9947#issuecomment-665182547 ，请试一下看看能不能帮到你。我也建议遇到问题先放狗 （Google），搞不好一下子就解决啦。

##### *东：
> Pod 版本更新那个地方的几个步骤不错，有启发

