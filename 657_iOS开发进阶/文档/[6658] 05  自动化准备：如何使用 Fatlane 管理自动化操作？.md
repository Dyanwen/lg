<p data-nodeid="1141" class="">要成为一个优秀的 iOS 开发者，我们要做的事情远多于“开发”，例如我们要构建和打包 App，管理证书，为 App 进行签名，把 App 分发给测试组，上传 App 到 App Store，等等。这些操作不但繁琐费时，而且容易出错。那么，有没有更便利的方法呢？有，那就是使用 fastlane 来完成这些重复性的工作。接下来这一讲，我们主要聊的也就是这个主题。</p>
<h3 data-nodeid="1142">fastlane 安装</h3>
<p data-nodeid="1143"><strong data-nodeid="1239">fastlane</strong> 是用 Ruby 语言编写的一个命令行工具，可以自动化几乎所有 iOS 开发所需要的操作，例如自动打包和签名 App，自动上传到 App Store 等等。有了 fastlane，我们就可以开发一套统一的、可靠的和可共享的配置，团队所有成员都可以通过这套配置实现自动化操作，减少重复性劳动。</p>
<p data-nodeid="1144">如何安装 fastlane 呢？我记得在第一讲就曾提到过，可以使用 Bundler 来安装，只需要在 Gemfile 文件里面添加以下一行即可：</p>
<pre class="lang-java" data-nodeid="1145"><code data-language="java">gem <span class="hljs-string">"fastlane"</span>, <span class="hljs-string">"2.166.0"</span>
</code></pre>
<p data-nodeid="1146">注意，由于是通过 Bundler 来安装 fastlane，每次执行 fastlane 命令前，都需要加上<code data-backticks="1" data-nodeid="1242">bundle exec</code>（<code data-backticks="1" data-nodeid="1244">bundle exec fastlane</code>）。不过为了简洁，在这里后面凡涉及 fastlane 命令时，我会省略<code data-backticks="1" data-nodeid="1246">bundle exec</code>部分。</p>
<h3 data-nodeid="1147">Action 与 Lane</h3>
<p data-nodeid="1148">fastlane 为我们提供了一百多个 Action，它们是 iOS 项目开发中所有自动化操作的基础。所谓的<strong data-nodeid="1254">Action</strong>，你可以理解成是 fastlane 自动化流程中的最小执行单元。一般常用的 Action 有：</p>
<ul data-nodeid="1149">
<li data-nodeid="1150">
<p data-nodeid="1151">scan，用于自动测试 App；</p>
</li>
<li data-nodeid="1152">
<p data-nodeid="1153">cert，用于自动生成和管理 iOS App 签名的证书；</p>
</li>
<li data-nodeid="1154">
<p data-nodeid="1155">sigh，用于自动生成、更新、下载和修复 Provisioning Profile；</p>
</li>
<li data-nodeid="1156">
<p data-nodeid="1157">match，为整个团队自动管理和同步证书和 Provisioning Profile；</p>
</li>
<li data-nodeid="1158">
<p data-nodeid="1159">gym，用于自动构建和打包 App；</p>
</li>
<li data-nodeid="1160">
<p data-nodeid="1161">snapshot，用于自动在不同设备上截屏；</p>
</li>
<li data-nodeid="1162">
<p data-nodeid="1163">pilot，用于自动把 App 部署到 TestFlight 并管理测试用户；</p>
</li>
<li data-nodeid="1164">
<p data-nodeid="1165">deliver，用于自动把 App 上传到 App Store；</p>
</li>
<li data-nodeid="1166">
<p data-nodeid="1167">pem，用于自动生成和更新推送消息的 Profile。</p>
</li>
</ul>
<p data-nodeid="1168">这些 Action 怎么执行呢？我们可以通过<code data-backticks="1" data-nodeid="1265">fastlane &lt;action&gt;</code>（例如<code data-backticks="1" data-nodeid="1267">fastlane scan</code>）来执行。下面是执行效果，它提示我选择其中一个 Scheme 来继续执行。</p>
<p data-nodeid="1169"><img src="https://s0.lgstatic.com/i/image6/M00/16/10/CioPOWBF-DaAV56WAAR8Ne2bQ4c230.png" alt="Drawing 0.png" data-nodeid="1271"></p>
<p data-nodeid="1170">从运行情况可知，尽管这些 Action 为我们提供了不少便利，但还是需要手工输入来继续。所以，我不推荐你直接使用这些 Action，而是根据项目需要，在开发自己的自动化操作时通过传入合适的参数来调用 fastlane 所提供的 Action。</p>
<p data-nodeid="1171">具体来说，我们可以把所需的 Action 组合在一起，开发出对应的自动化操作。在 fastlane 中，我们把这个自动化操作或者任务叫作 <strong data-nodeid="1281">Lane</strong>。<strong data-nodeid="1282">实际上， iOS 开发中的所有自动化操作，主要通过 Lane 来封装和配置的。</strong></p>
<p data-nodeid="1172"><img src="https://s0.lgstatic.com/i/image6/M00/16/10/CioPOWBF-EqACKcpAAC6PT8Ghi4740.png" alt="Drawing 2.png" data-nodeid="1285"></p>
<p data-nodeid="1173">Lane 和 Action 的关系如上图所示， 一条 Lane 可以通过参数调用一个或几个 Action 。以 Moments app&nbsp;为例，我们要自动打包和签名 App，那么我就建了一条名叫<code data-backticks="1" data-nodeid="1287">archive_appstore</code>的 Lane。因为这条 Lane 用到的“更新签名”和“打包”在 fastalne 里已经提供了相关的 Action——<code data-backticks="1" data-nodeid="1289">update_code_signing_settings</code>和<code data-backticks="1" data-nodeid="1291">gym</code>，我就可以到它官网去寻找，从而减轻了开发工作量。</p>
<p data-nodeid="1174">一般，iOS 项目所需的自动化操作都配置为 Lane 并保存在 Fastfile 文件，由 Git 统一管理起来，共享给所有成员。然后，大家就可以使用统一的自动化配置了。</p>
<p data-nodeid="1175">这里的 Fastfile 文件是怎么出来的呢？</p>
<p data-nodeid="1176">它是由<code data-backticks="1" data-nodeid="1296">fastlane init</code>命令自动生成。这条命令会建立一个 fastlane 文件夹，文件夹里除了 Fastfile ，还有 Appfile，以及执行过程中所生成的一些中间文件（如截图、日志与报告等）。因为我们之前已经在 .gitignore 文件里把这些中间文件忽略了，因此这些中间文件不再保存到 Git 里面。</p>
<p data-nodeid="1177">fastlane 文件夹里的 Appfile，用于保存 App 的唯一标识符和 Apple ID 等信息。当 fastlane 在执行一个 Action 的时候，首先会使用传递进来的参数，当参数没有传递进来时，fastlane 会从 Appfile 文件中查找并使用对应的信息。</p>
<p data-nodeid="1178">比如，我们在 Appfile 配置了<code data-backticks="1" data-nodeid="1300">app_identifier "com.ibanimatable.moments"</code>以后，在调用<code data-backticks="1" data-nodeid="1302">match</code>Action 时可以不传入<code data-backticks="1" data-nodeid="1304">app_identifier</code>参数，fastlane 会自动把<code data-backticks="1" data-nodeid="1306">"com.ibanimatable.moments"</code>作为<code data-backticks="1" data-nodeid="1308">app_identifier</code>的值进行调用。</p>
<p data-nodeid="1179">但是为了方便管理所有的 Lane ，保证每次执行的效果都一样，我建议在每次调用 Action 的时候，都明确传递每一个所需的参数，而不是从 Appfile 文件读取。下面我就演示下如何明确传递每一个参数来执行<code data-backticks="1" data-nodeid="1311">match</code>Action。</p>
<pre class="lang-java" data-nodeid="1180"><code data-language="java">  match(
&nbsp; &nbsp; &nbsp; type: "appstore",
&nbsp; &nbsp; &nbsp; force: true,
&nbsp; &nbsp; &nbsp; storage_mode: "git",
&nbsp; &nbsp; &nbsp; git_url: "https://github.com/JakeLin/moments-codesign",
&nbsp; &nbsp; &nbsp; app_identifier: "com.ibanimatable.moments", # pass  app_identifier explicitly
&nbsp; &nbsp; &nbsp; team_id: "6HLFCRTYQU",
&nbsp; &nbsp; &nbsp; api_key: api_key
  )
</code></pre>
<h3 data-nodeid="1181">常用 Lane 定义</h3>
<p data-nodeid="1182">通过上面的介绍你已经知道，我们会使用 Lane 来封装项目所需的各个自动化操作。那么，这些 Lane 是如何开发定义的呢？接下来，我就为你介绍几种非常实用的 Lane，一起来看看怎么做。</p>
<h4 data-nodeid="1183">扫描和检查代码</h4>
<p data-nodeid="1184">每条 Lane 的定义都是放在一个<code data-backticks="1" data-nodeid="1317">lane &lt;lane_name&gt; do &lt;lane_body&gt; end</code>的代码块里面。它以关键字<code data-backticks="1" data-nodeid="1319">lane</code>开头，接着是这条 Lane 的名字。 下面是用于检查代码的 Lane 源码。</p>
<pre class="lang-ruby" data-nodeid="1185"><code data-language="ruby">lane <span class="hljs-symbol">:lint_code</span> <span class="hljs-keyword">do</span>
  puts(<span class="hljs-string">"Lint code using SwfitLint"</span>)
  swiftlint(
    <span class="hljs-symbol">mode:</span> <span class="hljs-symbol">:lint</span>,
    <span class="hljs-symbol">executable:</span> <span class="hljs-string">"./Pods/SwiftLint/swiftlint"</span>,  <span class="hljs-comment"># Important if you've installed it via CocoaPods</span>
    <span class="hljs-symbol">config_file:</span> <span class="hljs-string">'./Moments/.swiftlint.yml'</span>,
    <span class="hljs-symbol">raise_if_swiftlint_error:</span> <span class="hljs-literal">true</span>)
<span class="hljs-keyword">end</span>
</code></pre>
<p data-nodeid="1186">在上面的例子中，我们定义了一个叫作<code data-backticks="1" data-nodeid="1322">lint_code</code>的 Lane。因为 fastlane 使用 Ruby 开发，所以在 Fastfile 里面，Lane 的名字也遵循它的编码规范，使用小写字母和下划线组合的蛇式命名法。</p>
<p data-nodeid="1187">Lane 的实现逻辑放在<code data-backticks="1" data-nodeid="1325">do</code>和<code data-backticks="1" data-nodeid="1327">end</code>中间，我们可以调用 fastlane 提供的任意 Action。在这个例子中我们就调用了<code data-backticks="1" data-nodeid="1329">swiftlint</code>Action，并把<code data-backticks="1" data-nodeid="1331">lint</code>传递给<code data-backticks="1" data-nodeid="1333">mode</code>参数，以此来执行代码扫描和检查操作。</p>
<p data-nodeid="1188">特别需要注意的是，由于我们之前使用了 CocoaPods 来安装 SwiftLint，因此要为<code data-backticks="1" data-nodeid="1336">executable</code>参数指定 SwiftLint 的安装路径<code data-backticks="1" data-nodeid="1338">./Pods/SwiftLint/swiftlint</code>。同时要把 .swiftlint.yml 文件的所在路径也传递给<code data-backticks="1" data-nodeid="1340">config_file</code>参数。这样就能保证 fastlane 使用了统一的 SwiftLint 版本和规则文件，方便团队所有人执行该 Lane 时得到统一的效果。</p>
<p data-nodeid="2287" class="te-preview-highlight">当一条 Lane 开发配置完毕以后，我们就可以在项目的根目录执行 <code data-backticks="1" data-nodeid="2289">fastlane &lt;lane_name&gt;</code>。比如扫描和检查代码的 Lane ，我们可以在终端输入<code data-backticks="1" data-nodeid="2291">fastlane lint_code</code>看到它的执行效果。</p>


<blockquote data-nodeid="1190">
<p data-nodeid="1191">Driving the lane 'ios lint_code'<br>
Lint code using SwfitLint<br>
--- Step: swiftlint ---<br>
$ ./Pods/SwiftLint/swiftlint lint --config ./Moments/.swiftlint.yml<br>
Linting Swift files in current working directory<br>
Linting 'Strings.swift' (1/87)<br>
Linting 'MomentListItemViewModel.swift' (2/87)<br>
Linting ......s</p>
<p data-nodeid="1192">UIButtonExtensions.swift:14:46: warning: no_hardcoded_strings Violation: Please do not hardcode strings and add them to the appropriate <code data-backticks="1" data-nodeid="1386">Localizable.strings</code> file; a build script compiles all strings into strongly typed resources available through <code data-backticks="1" data-nodeid="1388">Generated/Strings.swift</code>, e.g. `L10n.accessCamera (no_hardcoded_strings)<br>
Done linting! Found 6 violations, 0 serious in 87 files.<br>
fastlane.tools finished successfully</p>
</blockquote>
<p data-nodeid="1193">在执行过程中，fastlane 先从 Fastfile 文件里名叫<code data-backticks="1" data-nodeid="1403">lint_code</code>的 Lane 的定义，然后执行了该 Lane 里使用到的 swiftlint Action。swiftlint Action 会把项目下 87 个 Swift 源代码文件进行扫描和检查，并把所有不符合规范的代码提示给我们。</p>
<h4 data-nodeid="1194">格式化代码</h4>
<p data-nodeid="1195">检查代码之后，接下来就是清理不符合规范的代码，比如删掉所有代码中不必要的空格或者空行，修正缩进的大小等等。我们可以定义一条叫作<code data-backticks="1" data-nodeid="1407">format_code</code>的 Lane 来执行该功能。有了它以后，我们只需要执行<code data-backticks="1" data-nodeid="1409">fastlane format_code</code>就能把整个项目所有的代码进行格式化。</p>
<pre class="lang-ruby" data-nodeid="1196"><code data-language="ruby">lane <span class="hljs-symbol">:format_code</span> <span class="hljs-keyword">do</span>
  puts(<span class="hljs-string">"Lint and format code using SwfitLint"</span>)
  swiftlint(
    <span class="hljs-symbol">mode:</span> <span class="hljs-symbol">:autocorrect</span>,
    <span class="hljs-symbol">executable:</span> <span class="hljs-string">"./Pods/SwiftLint/swiftlint"</span>,  <span class="hljs-comment"># Important if you've installed it via CocoaPods</span>
    <span class="hljs-symbol">config_file:</span> <span class="hljs-string">'./Moments/.swiftlint.yml'</span>,
    <span class="hljs-symbol">raise_if_swiftlint_error:</span> <span class="hljs-literal">true</span>)
<span class="hljs-keyword">end</span>
</code></pre>
<p data-nodeid="1197"><code data-backticks="1" data-nodeid="1411">format_code</code>和<code data-backticks="1" data-nodeid="1413">lint_code</code>两条 Lane 都使用了<code data-backticks="1" data-nodeid="1415">swiftlint</code>Action，唯一不同的地方是为<code data-backticks="1" data-nodeid="1417">mode</code>参数传递了<code data-backticks="1" data-nodeid="1419">autocorrect</code>。</p>
<h4 data-nodeid="1198">排序 Xcode 项目文件列表</h4>
<p data-nodeid="1199">在多人开发的项目下，我们经常会修改项目文件，这往往很容易引起合并冲突，而合并 xcodeproj 文件又是一件非常麻烦的事情。怎么办呢？</p>
<p data-nodeid="1200">一个有效办法就是在每次新建源代码和资源文件时，把 xcodeproj 里面的文件列表进行重新排序。这样能极大地减低合并冲突的发生。我们把这一个经常使用到的操作也配置到 Fastfile 里面，如下所示。</p>
<pre class="lang-ruby" data-nodeid="1201"><code data-language="ruby">lane <span class="hljs-symbol">:sort_files</span> <span class="hljs-keyword">do</span>
  puts(<span class="hljs-string">"Sort the files for the Xcode project"</span>)
  sh <span class="hljs-string">"../scripts/sort-Xcode-project-file.pl ../Moments/Moments.xcodeproj"</span>
<span class="hljs-keyword">end</span>
</code></pre>
<p data-nodeid="1202">可以看到，fastlane 除了能调用其提供的 Action 以外，还可以通过<code data-backticks="1" data-nodeid="1425">sh</code>来调用其他程序命令。在这里我们调用了由苹果公司提供的一个<strong data-nodeid="1435">Perl 程序来为 xcodeproj 里面的文件列表进行排序</strong>。你也可以在<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/scripts/sort-Xcode-project-file.pl" data-nodeid="1433">拉勾教育的代码仓库</a>找到这个Perl 程序。</p>
<h4 data-nodeid="1203">调用其他 Lane 操作</h4>
<p data-nodeid="1204">除了调用一些程序命令（如<code data-backticks="1" data-nodeid="1438">sh</code>）以外，一条 Lane 还可以调用 Fastfile 里面其他的 Lane。例如我们定义了一条叫作<code data-backticks="1" data-nodeid="1440">prepare_pr</code>的 Lane ，它可以帮我们在提交 Pull Request 之前做一些必要的准备。下面这个代码表示的就是，这条 Lane 在内部调用了另外两条 Lane ——<code data-backticks="1" data-nodeid="1442">format_code</code>和<code data-backticks="1" data-nodeid="1444">sort_files</code>，以此来同时完成格式化代码和排序 Xcode 项目文件列表的操作。</p>
<pre class="lang-ruby" data-nodeid="1205"><code data-language="ruby">lane <span class="hljs-symbol">:prepare_pr</span> <span class="hljs-keyword">do</span>
  format_code
  sort_files
<span class="hljs-keyword">end</span>
</code></pre>
<h4 data-nodeid="1206">定义私有 Lane 和返回值</h4>
<p data-nodeid="1207">类似于 Swift 语言能通过<code data-backticks="1" data-nodeid="1448">private</code>来定义内部使用的方法，我们也能定义私有 Lane 给Fastfile 内的其他 Lane 所调用，提高代码的复用。其做法就是把原先的关键字<code data-backticks="1" data-nodeid="1450">lane</code>替换成<code data-backticks="1" data-nodeid="1452">private_lane</code>。例如我们定义一条叫作<code data-backticks="1" data-nodeid="1454">get_pi</code>的私有 Lane，代码如下。</p>
<pre class="lang-ruby" data-nodeid="1208"><code data-language="ruby">private_lane <span class="hljs-symbol">:get_pi</span> <span class="hljs-keyword">do</span>
  pi = <span class="hljs-number">3.1415</span>
  pi
<span class="hljs-keyword">end</span>
</code></pre>
<p data-nodeid="1209">该 Lane 的实现体有两行代码，第一行是给一个临时变量<code data-backticks="1" data-nodeid="1457">pi</code>赋值。第二行表示把<code data-backticks="1" data-nodeid="1459">pi</code>作为返回值传递给调用者。例如下面就演示了如何调用<code data-backticks="1" data-nodeid="1461">get_pi</code>并取得返回值。</p>
<pre class="lang-ruby" data-nodeid="1210"><code data-language="ruby">lane <span class="hljs-symbol">:foo</span> <span class="hljs-keyword">do</span>
  pi = get_pi
  puts(pi)
<span class="hljs-keyword">end</span>
</code></pre>
<p data-nodeid="1211">这是执行<code data-backticks="1" data-nodeid="1464">fastlane foo</code>的结果：</p>
<blockquote data-nodeid="1212">
<p data-nodeid="1213">Driving the lane 'ios foo'<br>
--- Step: Switch to ios get_pi lane ---<br>
Cruising over to lane 'ios get_pi'<br>
Cruising back to lane 'ios foo'<br>
3.1415</p>
</blockquote>
<p data-nodeid="1214">fastlane 首先调用<code data-backticks="1" data-nodeid="1495">foo</code>Lane，然后进去<code data-backticks="1" data-nodeid="1497">get_pi</code>Lane 并返回到<code data-backticks="1" data-nodeid="1499">foo</code>，同时把返回结果打印出来。</p>
<h3 data-nodeid="1215">总结</h3>
<p data-nodeid="1216">这一讲我介绍了如何从头开始搭建一个 fastlane 环境。在这里需要注意三点：</p>
<ol data-nodeid="1217">
<li data-nodeid="1218">
<p data-nodeid="1219">不要单独手工执行 fastlane 所提供的 Action，而是使用 Fastfile 文件来统一开发、配置和管理日常中经常使用的所有自动化操作；</p>
</li>
<li data-nodeid="1220">
<p data-nodeid="1221">在开发我们的 Lane 时，要优先使用和调用 fastlane 提供的 Action，因为这些 Action 都是经过社区完善的，且会随着 Xcode 版本的升级而更新；</p>
</li>
<li data-nodeid="1222">
<p data-nodeid="1223">当我们调用 fastlane 所提供的 Action 时，要明确传递各个参数，在执行过程中就无须任何手工交互就能从头到尾执行整个操作。</p>
</li>
</ol>
<p data-nodeid="1224"><img src="https://s0.lgstatic.com/i/image6/M00/16/13/Cgp9HWBF-HeACPAgAAQVQ3FLLec967.png" alt="Drawing 4.png" data-nodeid="1508"></p>
<p data-nodeid="1225">有了项目需要的所有 Lane 以后，能有效减轻团队成员的重复劳动，并为项目的自动化和工程化打下坚实的基础。在后面的章节中，我会详细介绍如何使用 fastlane 来管理证书，打包 App 和上传到 App Store。</p>
<p data-nodeid="1226">思考题：</p>
<blockquote data-nodeid="1227">
<p data-nodeid="1228">在你的项目中是怎样实施自动化的？如果你是第一次使用 fastlane，请按照上述的文章和 fastlane 的官方文档来开发一条执行测试的 Lane。</p>
</blockquote>
<p data-nodeid="1229">可以把你的答案写得留言区哦，下一讲我将介绍如何使用 Git 与 GitHub 统一代码管理流程。</p>
<p data-nodeid="1230"><strong data-nodeid="1516">源码地址：</strong></p>
<blockquote data-nodeid="1231">
<p data-nodeid="1232" class="">Fastfile 文件地址：<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/fastlane/Fastfile#L19-L50" data-nodeid="1520">https://github.com/lagoueduCol/iOS-linyongjian/blob/main/fastlane/Fastfile#L19-L50</a></p>
</blockquote>

---

### 精选评论

##### **东：
> 老师，对">xcodeproj 文件进行排序后，打开项目目录顺序很乱不习惯，这个你是怎么解决的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 奇怪呢，那个只会改变 xcodeproj 文件里面的文本，不会改变物理目录的顺序的呀。

##### *杨：
> 想请教大佬两个问题.1. fastlane如果我们要配置一些账号密码写在哪里比较安全呢? 我之前写在脚本里明文通过git管理感觉很不合理.2.看作者对脚本 以及工具进行了统一管理, 如果进行组件化多repo开发, 是不是使用git submodule进行管理方便同步一点呢?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 我建议使用 API Key，不要使用账户和密码。2.可以使用 Pod 呀，内部建立一个 Pod repo 就能在内部发布 Pod 版本，实用方只需更新 Podfile 就能引用内部 Pod。

##### **菲：
> 跟着操作了一遍，.swiftlint.yml 文件里面excluded:中，还要加入- "./vendor"、- "./Frameworks"，不然也会执行检查

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你说的是对的，Moments 没有 Vendor 文件夹，所以没有 exclude。而 Frameworks 里面放了 DesignKit。因为 DesignKit 也是我们的内部代码，我想同时检查 DesignKit 的代码风格，所以就没有 exclude。你可以根据自己项目的情况来添加或者忽略要检查的问题。

##### *明：
> 老师，执行fastlane init后，我跟着terminal提示操作，第一步选择用途：3(发布到App Store），第二步选择scheme：2（这个scheme：项目名称-appStore）第三步：输入账号和密码....最后提示Looks like the app 'com.xxx.xxx.debug' isn't available onApp Store Connect我在Project的info中，对应的configurations选的应该没错，debug对应debug,internal对应internal,appstore也一样。是哪里还有没删干净？target里的SigningCapablities里也对应3个，bundle Id也没问题。这个是哪里没处理对吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为这个 App 的 Bundle ID 我已经在 App Store Connect 上注册过了，你不能上传同一个 App，应该改一下 Bundle ID 就好啦

##### **泽：
> 使用sort-Xcode-project-file.pl 这个脚本总是报错$ sh sort-Xcode-project-file.pl ../Moments/Moments.xcodeprojsort-Xcode-project-file.pl: line 31: use: command not foundsort-Xcode-project-file.pl: line 32: use: command not foundsort-Xcode-project-file.pl: line 34: use: command not foundsort-Xcode-project-file.pl: line 35: use: command not foundsort-Xcode-project-file.pl: line 36: syntax error near unexpected token `('sort-Xcode-project-file.pl: line 36: `use File::Temp qw(tempfile);

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问你是通过 fastlane 来执行的吗？执行方式是在根目录下执行 bundle exec fastlane sort_files

##### **泽：
> 老师，swiftlint在编译的时候已经提示了规范检查的警告， 为什么还要使用fastlane的action，第二个问题是为什么xcodeproj 里面的文件列表进行重新排序可以降低合并冲突？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你的提问很好呀。关于第一个问题，在平常开发中我们确实不需要使用 fastlane 来进行检查，但是可以使用 fastlane 来格式化有问题的代码呀。同时有了 fastlane 我们就可以在 CI 里面进行自动化检测，代码规范不通过的 PR 就不允许合并了，有些保证整个团队代码风格的一致性。关于第二个问题是这样的，如果我们不进行排序，当两个 PR 都修过了 Xcode 项目文件时发生冲突的机会会增大，如果你合并过项目文件就知道，合并过程很容易搞错的。如果排序了，只有两个 PR 修过了名字十分相似的文件才会有冲突，哪怕有冲突，合并也相对简单很多了。

##### **彬：
> 突然看完就感觉，不学好Ruby，是没法做好项目自动化管理的😤

##### **泽：
> fastlane和jenkinsyou什么关系？老师

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; fastlane 是 自动化工具，而 Jenkins 是 CI 服务，CI 通常需要自动化工具例如 fastlane 来实现自动构建等操作。我们会在第四部详细讲述 fastlane 和 CI。到时候留意一下哦。

##### *朋：
> 仔细看了下 目录 是我看错。 不好意思。 我以为.swiftlint.yml 这个也是在项目 根目录下

##### *朋：
> Fastlane 中一个lint_code 的命令中的 config_file 这路径是不是有问题？ 应该没有 ./Moments 吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有问题哦，执行bundle exec fastlane lint_code 命令是在项目的根目录下，会打印出， $ ./Pods/SwiftLint/swiftlint lint --config ./Moments/.swiftlint.yml

##### **坚：
> 打卡，学习Ruby充分性😄

##### **坚：
> 打卡，学习Ruby充分性😄

##### *召：
> 现在项目里用的是Jenkins和脚本自动打包

##### **辉：
> 沙发~

