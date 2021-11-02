<p data-nodeid="10665" class="">在软件开发领域有很多有趣且重要的话题，比如使用什么样的系统架构来让代码更容易维护，使用哪些第三方库能提高开发效率，等等。但也有一些话题不仅无趣，还很难得出结论，比如像下面这行变量定义，里面的空格哪个正确？</p>
<pre class="lang-java" data-nodeid="10666"><code data-language="java">let name: String = <span class="hljs-string">"Jake"</span>
let name : String = <span class="hljs-string">"Jake"</span>
let name :String = <span class="hljs-string">"Jake"</span>
let name: String= <span class="hljs-string">"Jake"</span>
let name: String=<span class="hljs-string">"Jake"</span>
</code></pre>
<p data-nodeid="10667">还有代码缩减，到底是用 2 个空格还是 4 个？这就像豆浆到底是喝甜的还是喝咸的一样，并没有标准答案。也因此，出现了许多永无休止的讨论。特别是当新成员所提交的代码风格，与团队其他成员有很大的区别时，往往会出现沟通与协作问题，甚至发生争执而影响工作。此时，团队如果有一套统一的编码规范，那么这样的问题就很容易解决。</p>
<p data-nodeid="10668">除了能促进沟通协作，一套统一的编码规范还能降低代码维护的成本和减少 Bug 的数量。此外，由于规范往往由团队资深开发者指定并不断完善，也有助于其他团队成员快速成长。</p>
<p data-nodeid="10669">既然统一的编码规范由那么多优点，那么我们如何在团队中实施统一编码规范呢？在 iOS 开发领域，使用 SwiftLint 能有效地建立和改进 Swift 项目的编码规范。接下来我就和你聊聊这方面的内容。</p>
<h3 data-nodeid="10670">安装 SwiftLint</h3>
<p data-nodeid="10671">安装 SwiftLint 的方式有很多种，例如使用 Homebrew，Mint，下载 SwiftLint.pkg 安装包等等。但我只推荐 CocoaPods 这一种方法，因为通过 CocoaPods 可以有效地管理 SwiftLint 的版本，从而保证团队内各个成员都能使用一模一样的 SwiftLint 及其编码规范。</p>
<p data-nodeid="10672">通过 CocoaPods 来安装 SwiftLint 非常简单。在 Moments App 项目中，我们在<code data-backticks="1" data-nodeid="10736">Podfile</code>文件中添加<code data-backticks="1" data-nodeid="10738">SwiftLint</code>Pod 即可。</p>
<pre class="lang-java" data-nodeid="10673"><code data-language="java">pod <span class="hljs-string">'SwiftLint'</span>, <span class="hljs-string">'= 0.41.0'</span>, configurations: [<span class="hljs-string">'Debug'</span>]
</code></pre>
<p data-nodeid="10674">由于我们只在开发环境下使用 SwiftLint，因此配置了只有<code data-backticks="1" data-nodeid="10741">Debug</code>的 Build Configuration 才生效。</p>
<p data-nodeid="10675">为了每次编译完都使用 SwiftLint 来检查代码，我们需要在主 App Target<strong data-nodeid="10754">Moments</strong>的 Build Phases 里面添加<strong data-nodeid="10755">Run SwiftLint</strong>步骤。然后配置它执行<code data-backticks="1" data-nodeid="10752">"${PODS_ROOT}/SwiftLint/swiftlint"</code>命令。</p>
<p data-nodeid="10676"><img src="https://s0.lgstatic.com/i/image6/M00/0F/0F/CioPOWA9Et6AK5ZLAAKntpgVJ2o333.png" alt="Drawing 0.png" data-nodeid="10758"></p>
<p data-nodeid="10677">这里要注意，由于 SwiftLint 的设计是检查有效的 Swift 代码（编译通过的代码就是有效的代码），我们需要把<strong data-nodeid="10768">Run SwiftLint</strong>步骤放在<strong data-nodeid="10769">Compile Source</strong>步骤之后。否则 SwiftLint 可能会反馈一些错误的结果。</p>
<p data-nodeid="10678">有了上面的配置以后，每次编译程序， SwiftLint 都会自动执行检查，我们可以在 Xcode 上修正这些警告信息来保证编码规范的统一。</p>
<p data-nodeid="10679">￼<img src="https://s0.lgstatic.com/i/image6/M01/11/28/CioPOWA_X36AaLBrAAH6eimbZ_o003.png" alt="图片1.png" data-nodeid="10774"></p>
<p data-nodeid="10680">例如上面的截图所示，SwiftLint 告诉我们空格的使用不正确。</p>
<p data-nodeid="10681">那么，这些警告信息到底怎样来的呢？我们一起看看<code data-backticks="1" data-nodeid="10777">.swiftlint.yml</code>文件吧。</p>
<h3 data-nodeid="10682">.swiftlint.yml 文件</h3>
<p data-nodeid="10683">当我们执行 SwiftLint 命令时，它会自动帮我们启动一堆编码规则，并扫描和检查我们的项目。这些规则有<code data-backticks="1" data-nodeid="10781">comma</code>（逗号前后的空格处理），<code data-backticks="1" data-nodeid="10783">private_over_fileprivate</code>（优先使用 priviate），<code data-backticks="1" data-nodeid="10785">force_cast</code>（避免强制转型）等等 。详细规则列表你也可以在<a href="https://realm.github.io/SwiftLint/rule-directory.html" data-nodeid="10789">SwiftLint 官网</a>找到。</p>
<p data-nodeid="10684">但正如 SwiftLint 的作者所说： “规则存在，但并不意味着你必须用它”。我们需要根据团队自身的情况和成员的统一意见，来决定需要启动和关闭哪些规则。此时，就需要用到 .swiftlint.yml 文件了。</p>
<p data-nodeid="10685"><strong data-nodeid="10796">.swiftlint.yml</strong>主要用于启动和关闭 SwiftLint 所提供的规则，以及自定义配置与规则。一旦我们有了 .swiftlint.yml 文件以后，SwiftLint 在执行过程中会严格按照该文件的定义来扫描和检查代码。由于 .swiftlint.yml 是一个纯文本文件，我们可以通过 Git 统一管理，这样能保证整个团队在执行 SwiftLint 的时候都会得到一模一样的效果，从而保证了整个团队代码规范的一致性。</p>
<h3 data-nodeid="10686">规则设置</h3>
<p data-nodeid="10687">SwiftLint 提供了<code data-backticks="1" data-nodeid="10799">disabled_rules</code>,<code data-backticks="1" data-nodeid="10801">opt_in_rules</code>和<code data-backticks="1" data-nodeid="10803">only_rules</code>三种规则设置方法。其中，￼￼<code data-backticks="1" data-nodeid="10805">disabled_rules</code>能帮我们关闭默认生效的规则，而<code data-backticks="1" data-nodeid="10807">opt_in_rules</code>可以启动默认关闭的规则。</p>
<p data-nodeid="10688">另外，SwiftLint 所提供的每条规则都有一个叫作<strong data-nodeid="10818">Enabled by default</strong>的属性来表示该规则是否默认启动。例如<code data-backticks="1" data-nodeid="10814">class_delegate_protocol</code>规则是默认启动的，而<code data-backticks="1" data-nodeid="10816">array_init</code>规则是默认关闭的。</p>
<pre class="lang-java" data-nodeid="10689"><code data-language="java">disabled_rules:
  - class_delegate_protocol
opt_in_rules:
  - array_init
</code></pre>
<p data-nodeid="10690">上面的配置表示，关闭默认生效的<code data-backticks="1" data-nodeid="10820">class_delegate_protocol</code>，并同时启动<code data-backticks="1" data-nodeid="10822">array_init</code>。</p>
<p data-nodeid="10691">虽然使用<code data-backticks="1" data-nodeid="10825">disabled_rules</code>和<code data-backticks="1" data-nodeid="10827">opt_in_rules</code>能够完成配置，但我不推荐你使用它们&nbsp;，而是用<code data-backticks="1" data-nodeid="10829">only_rules</code>来定义每条生效的规则。</p>
<p data-nodeid="10692">我们在 Moments App 项目中也使用了<code data-backticks="1" data-nodeid="10832">only_rules</code>。你可以在<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Moments/.swiftlint.yml" data-nodeid="10836">拉勾教育的代码仓库</a>找到该 .swiftlint.yml 文件来查看项目启动的所有规则。由于<code data-backticks="1" data-nodeid="10838">only_rules</code>是 SwiftLint 0.41.0 引入的，如果你需要以前版本，可以使用<code data-backticks="1" data-nodeid="10840">whitelist_rules</code>来替代。下面是 .swiftlint.yml 文件中的部分规则。</p>
<pre class="lang-java" data-nodeid="10693"><code data-language="java">only_rules:
  - array_init
  - attributes
  - block_based_kvo
  - class_delegate_protocol
  - closing_brace
</code></pre>
<p data-nodeid="10694">通过<code data-backticks="1" data-nodeid="10843">only_rules</code>，我们可以把每一条规则明确添加到 SwiftLint 里面。这样能保证我们整个团队都使用一致的规则，而不会像使用<code data-backticks="1" data-nodeid="10845">disabled_rules</code>和<code data-backticks="1" data-nodeid="10847">opt_in_rules</code>那样，随着 SwiftLint 默认规则的改变，导致最终启动的规则不一样。</p>
<h4 data-nodeid="10695">自定义配置</h4>
<p data-nodeid="10696">在我们配置一条规则的时候，为了符合团队自身的情况，可以修改其默认配置。例如<code data-backticks="1" data-nodeid="10851">line_length</code>的默认配置是当一行代码多于 120 个字符的时候会报告编译警告，而多于 200 个字符的时候报告编译错误。</p>
<p data-nodeid="10697"><img src="https://s0.lgstatic.com/i/image6/M01/11/2B/Cgp9HWA_X6yAHMc8AAHVx2uT2fY153.png" alt="图片2.png" data-nodeid="10855"></p>
<div data-nodeid="10698"><p style="text-align:center">来源：SwiftLintFramework Docs</p></div>
<p data-nodeid="10699">我们可以在 .swiftlint.yml 文件中修改这些配置。</p>
<pre class="lang-java" data-nodeid="10700"><code data-language="java">line_length: <span class="hljs-number">110</span>
file_length:
  warning: <span class="hljs-number">500</span>
  error: <span class="hljs-number">1200</span>
</code></pre>
<p data-nodeid="10701">上述的配置表示我们修改了<code data-backticks="1" data-nodeid="10858">line_length</code>的配置，当一行代码多于 110 个字符（而不是默认的 120 个字符）时就会报告编译警告。我们也可以同时覆盖编译警告和编译错误的配置，例如把<code data-backticks="1" data-nodeid="10860">file_length</code>的编译警告改成 500，而编译错误改成 1200。</p>
<h4 data-nodeid="10702">自定义规则</h4>
<p data-nodeid="10703">除了 SwiftLint 所提供的默认规则以外，我们还可以自定义规则。例如在 Moments App 项目中，我就自定义了“不能硬编码字符串”的规则，具体如下：</p>
<pre class="lang-java" data-nodeid="10704"><code data-language="java">custom_rules:
  no_hardcoded_strings:
    regex: <span class="hljs-string">"([A-Za-z]+)"</span>
    match_kinds: string
    message: <span class="hljs-string">"Please do not hardcode strings and add them to the appropriate `Localizable.strings` file; a build script compiles all strings into strongly typed resources available through `Generated/Strings.swift`, e.g. `L10n.accessCamera"</span>
    severity: warning
</code></pre>
<p data-nodeid="10705">该规则<code data-backticks="1" data-nodeid="10865">no_hardcoded_strings</code>会通过正则表达式来检查字符串是否进行了硬编码。如果是SwiftLint 会根据我们的自定义规则显示警告信息，如下图所示。</p>
<p data-nodeid="10706"><img src="https://s0.lgstatic.com/i/image6/M00/11/2C/Cgp9HWA_X_2AWg7XAAJqf2s12IA729.png" alt="图片4.png" data-nodeid="10869"></p>
<h4 data-nodeid="10707">排除扫描文件</h4>
<p data-nodeid="10708">默认情况下 SwiftLint 会扫描和检查整个项目的所有代码。因为一些第三方依赖库的源码风格可能和我们团队的风格不一致，为了方便使用第三方依赖库，我们可以用<code data-backticks="1" data-nodeid="10872">excluded</code>来把它排除在外，避免扫描和检查。示例如下：</p>
<pre class="lang-java" data-nodeid="10709"><code data-language="java">excluded:
  - Pods
</code></pre>
<p data-nodeid="10710">现在我们已经通过配置 .swiftlint.yml 文件来帮助我们统一编码规范了。</p>
<h3 data-nodeid="10711">总结</h3>
<p data-nodeid="10712">在这一讲，我介绍了如何使用 SwiftLint 来统一编码规范。特别是其中的<code data-backticks="1" data-nodeid="10877">only_rules</code>，我们要使用它来定义需要生效的规则。</p>
<p data-nodeid="10713"><img src="https://s0.lgstatic.com/i/image6/M00/11/29/Cgp9HWA_XxiAF_9EAAJXiOcRtSY049.png" alt="思维导图+二维码.png" data-nodeid="10881"></p>
<p data-nodeid="10714">此外，在制定编码规范时，我们还需要注意以下几点。</p>
<p data-nodeid="10715">首先，所制定的规范要和业界标准同步，这能让新成员接手代码时，更容易接受而不是反驳。一个建议是，你可以从 SwiftLint 所提供的默认规则开始，毕竟这些规则都是通过许多人沟通和完善下来的，比你独自一人想出来要靠谱得多。</p>
<p data-nodeid="10716">其次，在制定规范时，重点是提高代码的可读性，而不是为了高大上去使用黑魔法或者选择那些不常用功能等。这样可以让团队绝大部分成员更容易理解和遵循这些规范。</p>
<p data-nodeid="10717">最后要强调的是，一套编码规范是需要不断迭代和完善的，我建议团队要定时 Retro（Retrospective，敏捷回顾），讨论和优化这些规范，让得大家都有机会贡献到规范中，增加他的认同感。这种做法在多团队平行开发的环境下特别有效。</p>
<p data-nodeid="10718">思考题：</p>
<blockquote data-nodeid="10719">
<p data-nodeid="10720">你所做的团队除了使用 SwiftLint 等工具检查以外，还使用了哪些手段来保证编码规范的统一呢？</p>
</blockquote>
<p data-nodeid="10721">请把回答写到下面的留言区哦，下一讲我将介绍如何使用 Fastlane 执行自动化操作。</p>
<p data-nodeid="10722"><strong data-nodeid="10892">源码地址：</strong></p>
<blockquote data-nodeid="10723">
<p data-nodeid="10724">swiftlint.yml 文件<br>
<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Moments/.swiftlint.yml" data-nodeid="10897">https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Moments/.swiftlint.yml</a></p>
</blockquote>
<hr data-nodeid="10725">
<p data-nodeid="10726"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="10902"><img src="https://s0.lgstatic.com/i/image6/M00/08/77/Cgp9HWA0wqWAI70NAAdqMM6w3z0673.png" alt="Drawing 1.png" data-nodeid="10901"></a></p>
<p data-nodeid="10727"><strong data-nodeid="10906">《大前端高薪训练营》</strong></p>
<p data-nodeid="10728" class="te-preview-highlight">12 个月打磨，6 个月训练，优秀学员大厂内推，<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="10910">点击报名，高薪有你</a>！</p>

---

### 精选评论

##### **通：
> 每次更新能多更新点吗，2节也可以啊，一节感觉不够看的😂

 ###### &nbsp;&nbsp;&nbsp; 官方客服回复：
> &nbsp;&nbsp;&nbsp; 因为老师是边写边上线更新的，为了防止断更，咱们专栏是一周两更，本专栏是每周二和周四更新，如果觉得了解更多，还可以查看咱们专栏的源码仓库，了解下整个新项目

##### **钦：
> oc 有什么好的代码规范检查工具么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以使一下 oclint，而且 fastlane 也支持。

##### **龙：
> 我看到有swiftlint autocorrect这个命令，但是目前swiftlint是使用cocoapods进行安装的，所以路径会在 项目根目录/pods/swiftlint/swiftlint，这种如果多个项目的话，使用起来还得现找路径然后在执行，有没有什么小tips，技巧可用？

##### **龙：
> 请问, 配置好以后, 如何每一次 run 的时候,自动执行"swiftlint autocorrect"吗?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦，这个可以 Build Phases 上加一个 step 执行。

##### *俊：
> xcode项目本地没有swiftlint.xml文件好象一样有效果，怎么做到的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你说的是 swiftlint.yml 文件吧，我在文件里面讲过，假如在项目里面不指定 swiftlint.yml 文件， SwiftLint 会使用一些默认的规则。但是推荐自己配置 swiftlint.yml 文件来保证整个团队都使用一样的规则。

##### **彬：
> 使用 SwiftLint 之后，连新建文件默认产生的 class 定义行 `{` 下面一行留空行都会警告...... 是真的让人无语，是不是过滤没设置好；提示：`Vertical Whitespace after Opening Braces Violation: Don't include vertical whitespace (empty line) after opening braces. (vertical_whitespace_opening_braces)` 这样的错误😰

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 假如你需要支持 `{` 下面一行空行，可以把这个规则去掉，但是我偏向于保留，因为代码更紧凑。还有，文章主要给出了如何实现规范的思路，并不是要求你严格执行这些规则。具体规范可按照你们团队的具体情况而定。

##### iOS：
> 少打了几个字。。。😭moments的yml不在swiftlint执行的文件夹内，我理解就是不在执行路径上。如何起作用的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在 fastlane 里面，我们通过了 config_file 来指定 .swiftlint.yml 文件的路径。在 Xcode 里面，默认会读取 Moments.xcodeproj 文件夹所在同一层的 .swiftlint.yml 文件。也可以在 --config来让 Xcode 搜索其他目录。

##### **彤：
> 可以出一个appcode 的相关使用教程吗？ 有些功能还不是很会用

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦，AppCode 是一个 IDE，打开的时候他会给一些提示，可以看一下。用法和 IntelliJ IDEA 以及 Android Studio 基本一样，可以 Google 相关文档看看。

##### iOS：
> . yml文件按照官方仓库的提示，在Swiftlint执行的路径添加，不起作用。而moments这个yml不在SwiftLint执行的文件，怎么操作？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我不是很理解你的问题，请问你有没有按照文章的步骤来添加 SwiftLint，我们使用的是 CocoaPods 的 SwiftLint，所以不是很明白 “moments这个yml不在SwiftLint执行的文件”。Moments App 的 .swiftlint.yml 放在 Moments/Moments 文件夹下面。如果你使用 VS Code 等工具就能看到。

##### **祥：
> 可以详细说一下老项目混编转Swift的可行方案吗，现在网上搜到的都是些没意义的流水文，我实际上操作起来有很多不兼容的地方，Swift能支持OC的特性，但是OC不支持Swift的特性。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我们在 Swift 1.1 的时候开始混编，我很明白你的痛苦之处呀，坑最多的是正如你说的 Bridging。要讲需要一个课程呀，我们后来的一个策略是尽量不要动 OC 的任何代码（除了新增 Nullability 那些供 Swift 使用以外），新功能全部使用 Swift 来写并慢慢替换 OC 的代码。然后尽量不用从 OC 引用 Swift 的代码，如果循环引用，以后代码很难维护，而且为了兼容性， Swift 的代码也学得像 OC。

##### **4349：
> 老师有没有班级微信群,拉一个一起学习

 ###### &nbsp;&nbsp;&nbsp; 官方客服回复：
> &nbsp;&nbsp;&nbsp; 可以通过课程下方的“添加专属社群”来加入班级社群哈~

##### **西瓜：
> 怎么让SwiftLint也检查Designkit

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我给你一个提示吧，使用的是 included，注意路径的选择，如果你做出来，来个 PR？

##### **0593：
> 短！

##### **泽：
> 老师您好，是不是OC使用OCLint

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是呀，但是我现在已经不怎样写 OC 了，以前写 OC 的时候，我使用 AppCode，基本上生成的代码都很标准了。

