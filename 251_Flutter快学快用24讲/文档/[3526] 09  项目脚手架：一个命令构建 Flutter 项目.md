<p data-nodeid="39710" class="">在基础功能部分，我已经讲解了从基础到规范的应用，接下来我们进入项目实战部分。本课时将介绍项目基础框架，并且应用脚手架功能实现轻松的初始化项目。</p>


<h3 data-nodeid="39019">项目基础框架</h3>
<p data-nodeid="39020">先看项目基础框架，我们将项目基础框架分为三个部分：核心代码部分、基础工具以及单元测试。</p>
<h4 data-nodeid="39021">核心代码</h4>
<p data-nodeid="40622">核心代码主要是在 lib 目录下，我们将 lib 下的各个功能进行了整理，可以用图 1 来表示各个模块之间的关系。</p>
<p data-nodeid="40623" class=""><img src="https://s0.lgstatic.com/i/image/M00/2E/C3/Ciqc1F8Flb-AGmmvAADAcKsYMc8004.png" alt="image.png" data-nodeid="40627"><br>
图 1 lib 核心目录结构</p>




<ul data-nodeid="39025">
<li data-nodeid="39026">
<p data-nodeid="39027">入口文件，main.dart 核心入口文件；</p>
</li>
<li data-nodeid="39028">
<p data-nodeid="39029">pages 作为具体的页面结构，可以通过 main.dart 直接加载，大部分还是通过 router.dart 进行跳转，pages 可以按照业务功能划分文件夹；</p>
</li>
<li data-nodeid="39030">
<p data-nodeid="39031">pages 下是各个组件组建而成，组件部分可以按照通用、基础和业务来划分；</p>
</li>
<li data-nodeid="39032">
<p data-nodeid="39033">组件中包含了样式、交互和数据三个部分，因此分别需要 styles 和 model 文件夹；</p>
</li>
<li data-nodeid="39034">
<p data-nodeid="39035">model 大部分数据来自服务端，因此需要一个 api 文件夹来与服务端交互；</p>
</li>
<li data-nodeid="39036">
<p data-nodeid="39037">类型校验部分贯穿整个项目，在 pages 、widgets 、 model 和 api 中都可能会被应用到。</p>
</li>
</ul>
<p data-nodeid="39038">按照上面的划分，唯一需要注意的是，各个目录下的二级目录需要根据你们自身的业务功能去设计。因为业务模块不一定是一个页面，在项目初期就应该按照业务模块规划好目录结构，后期维护成本会降低很多，同时提升可扩展性。</p>
<p data-nodeid="41538">例如 pages 需要三个页面，一个是首页内容，一个是用户个人信息页面，另外一个则是用户信息修改页面，那么我们可以按照表格 1 这样命名文件以及类。</p>
<p data-nodeid="42794" class=""><img src="https://s0.lgstatic.com/i/image/M00/2E/C3/Ciqc1F8FldOAPQ-gAABVfTIEj5I407.png" alt="image (1).png" data-nodeid="42801"><br>
表格 1 pages 业务划分目录结构</p>







<p data-nodeid="43562">widgets 下则与 pages 目录结构保持一致即可，model 、api 以及 struct 则需要根据的服务端协议的业务功能来定义目录结构。使用上面的目录方式，我们创建出了如图 2 所示的一个结构，提供大家参考。</p>
<p data-nodeid="43563" class=""><img src="https://s0.lgstatic.com/i/image/M00/2E/CF/CgqCHl8FleuAcV39AABWvTrY5U8584.png" alt="image2.png" data-nodeid="43567"><br>
图 2 项目目录结构示例</p>




<h4 data-nodeid="39069">基础工具</h4>
<p data-nodeid="39070">按照我们前面课时所设计的一些基础规范，这里需要两个基础的工具 dartfmt 和 dartanalyzer。将这两个工具整合在一起，一个为 shell 脚本和一个为 bat 脚本，整合后的文件叫作 format_check.sh 和 format_check.bat，里面包含以下代码。</p>
<p data-nodeid="39071">format_check.sh，该脚本主要适用于 Mac 系统和 Linux 系统。</p>
<pre class="lang-shell" data-nodeid="39072"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash">!/bin/bash</span>
<span class="hljs-meta">#</span><span class="bash"> 代码美化</span>
dartfmt -w --fix lib/
<span class="hljs-meta">#</span><span class="bash"> 代码规范检查</span>
dartanalyzer lib
<span class="hljs-meta">#</span><span class="bash"> 单元测试通过</span>
flutter test
</code></pre>
<p data-nodeid="39073">format_check.bat，该脚本主要适用于 Windows 系统。</p>
<pre class="lang-sql" data-nodeid="50655"><code data-language="sql">dartfmt -w <span class="hljs-comment">--fix lib/</span>
dartanalyzer lib
flutter test
</code></pre>



















<p data-nodeid="51038">在项目开发阶段只需要通过该命令来运行，就可以确保我们的一些基础规范是满足的，其他逻辑部分还是需要各个团队自身的 Code Review。</p>
<h4 data-nodeid="51039">单元测试</h4>


<p data-nodeid="51609">为了保证代码的健壮性，还需要生成对应的单元测试目录。针对上面的 lib 结构，我们生成对应的目录结构即可，唯一需要去掉的就是 styles 目录，例如，初始化的时候，我们对应生成下图 3 的目录层级结构。</p>
<p data-nodeid="51610" class=""><img src="https://s0.lgstatic.com/i/image/M00/2E/C3/Ciqc1F8Flg2AZvgFAABbTxGj0PU912.png" alt="image (2).png" data-nodeid="51618"><br>
图 3 单元测试目录结构</p>




<p data-nodeid="39080">以上部分就是整个项目框架的基础结构，接下来我们将这个基础的项目结构做成一个框架模版，使用脚手架的方式统一来创建和运行。</p>
<h3 data-nodeid="39081">脚手架应用</h3>
<p data-nodeid="39082">为了能够更好地体验，我们可以封装好这些一样的功能，开发出一个脚手架方式。前端同学会比较熟悉，将大部分初始化或者脚本化的功能统一封装成一个脚手架，通过脚手架执行项目的初始化。</p>
<h4 data-nodeid="39083">环境要求</h4>
<p data-nodeid="39084">这里需要使用到 Node.js 和 npm 环境，如果是前端开发，应该已具备。如果是非前端开发或者未安装相应 Node.js 和 npm 环境的可以前往<a href="https://nodejs.org/en/download/" data-nodeid="39188">官网下载安装</a>最新的版本即可。</p>
<h4 data-nodeid="39085">flutter-pro-cli</h4>
<p data-nodeid="39086">flutter-pro-cli，该工具可以轻松帮你完成项目框架结构的初始化，在安装完成上面的运行环境后，在命令运行窗口，运行下面的命令。</p>
<pre class="lang-sql" data-nodeid="56703"><code data-language="sql">npm <span class="hljs-keyword">install</span> -g flutter-pro-cli
</code></pre>













<p data-nodeid="39088">安装完成后，运行如下命令查看具体包含的功能。</p>
<pre class="lang-sql" data-nodeid="57094"><code data-language="sql">flutter-pro-cli -h
</code></pre>

<p data-nodeid="39090">可以看到如下的窗口提示信息。</p>
<pre class="lang-sql" data-nodeid="57485"><code data-language="sql">Usage: flutter-pro-cli [options] [command]
Options:
&nbsp; -h, <span class="hljs-comment">--help&nbsp; &nbsp; &nbsp; display help for command</span>
Commands:
&nbsp; init|i&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Generates new flutter project
&nbsp; <span class="hljs-keyword">check</span>|c&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">Check</span> the <span class="hljs-keyword">project</span> lib <span class="hljs-keyword">format</span>
&nbsp; run|r [<span class="hljs-keyword">check</span>]&nbsp; &nbsp;<span class="hljs-keyword">Check</span> the <span class="hljs-keyword">project</span> lib <span class="hljs-keyword">format</span> <span class="hljs-keyword">and</span> run
&nbsp; <span class="hljs-keyword">sync</span>-<span class="hljs-keyword">test</span>|st&nbsp; &nbsp; Generates <span class="hljs-keyword">new</span> <span class="hljs-keyword">test</span> <span class="hljs-keyword">path</span> base <span class="hljs-keyword">on</span> lib <span class="hljs-keyword">path</span>
&nbsp; <span class="hljs-keyword">help</span> [command]&nbsp; display <span class="hljs-keyword">help</span> <span class="hljs-keyword">for</span> command
</code></pre>

<ul data-nodeid="39092">
<li data-nodeid="39093">
<p data-nodeid="39094"><strong data-nodeid="39204">init</strong>，该操作会初始化好目录结构，包含 lib 和 test 目录下，其次会生成一个比较简单的 main.dart 和 router.dart 文件，并将我们需要的 check_format.sh 、 check_format.bat 以及 analysis_options.yaml 这三个文件放在项目根目录下。</p>
</li>
<li data-nodeid="39095">
<p data-nodeid="39096"><strong data-nodeid="39213">check</strong>，该操作执行  check_format.sh 或者 check_format.bat 文件来美化代码结构，并检查当前项目的代码是否符合我们规范。</p>
</li>
<li data-nodeid="39097">
<p data-nodeid="39098"><strong data-nodeid="39220">run</strong>，启动运行项目，可以带 check 参数执行 check_format.sh 先校验是否符合规范，符合则启动，否则不启动项目。这里的 run 要注意，需要优先打开手机模拟器，不然无法启动。</p>
</li>
<li data-nodeid="39099">
<p data-nodeid="39100"><strong data-nodeid="39225">sync-test</strong>，同步测试代码结构，为了减少大家写单元测试的时间，脚手架提供了方法，可以读取你项目代码文件，并且添加了一个最基础的测试，其他部分则需要自己补充。</p>
</li>
</ul>
<p data-nodeid="39101">我在实际项目开发过程中发现写测试用例确实挺麻烦，为了节省时间，可以针对性生成一些基础的测试代码用例，例如上面的 sync-test 会为我们创建好相应的目录结构，以及相应的测试代码文件。</p>
<h4 data-nodeid="39102">脚手架实现</h4>
<p data-nodeid="39103">脚手架使用的是 Node.js 来实现的，大家可以参考 <a href="https://github.com/love-flutter/flutter-pro-cli" data-nodeid="39231">gtihub 源码</a>，并在这基础上进行协同开发。由于使用的是 Node.js 来实现，这里就不过多介绍其实现原理，如果大家有兴趣可以进一步在 github 进行交流，希望共同完善这个脚手架。</p>
<h3 data-nodeid="39104">实战初始化</h3>
<p data-nodeid="39105">现在我们使用以上脚手架来初始化一个 Flutter 项目。首先第一步是创建项目 two you friend ，需要在 Android Studio 中创建好 Flutter 项目，项目创建完成后，在项目根目录打开命令行窗口，执行以下命令进行初始化。</p>
<pre class="lang-sql" data-nodeid="57876"><code data-language="sql">flutter-pro-cli init
</code></pre>

<p data-nodeid="39107">执行完该初始化成功后，打开手机模拟器运行下面的命令检查代码规范，并且启动项目。</p>
<pre class="lang-sql" data-nodeid="58267"><code data-language="sql">flutter-pro-cli run <span class="hljs-keyword">check</span>
</code></pre>

<p data-nodeid="39109">为了尝试自动化生成测试代码，我们可以在项目中的 lib/pages/home_page/ 目录下创建一个 index.dart 。然后再运行下面的命令。</p>
<pre class="lang-sql" data-nodeid="58658"><code data-language="sql">flutter-pro-cli st
</code></pre>

<p data-nodeid="39111">运行完后，在相应的 test/pages/home_page 目录下你将看到 index_test.dart 文件，里面将包含下面的测试代码。</p>
<pre class="lang-dart" data-nodeid="39112"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter_test/flutter_test.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/pages/home_page/index.dart'</span>;
<span class="hljs-comment">// @todo</span>
<span class="hljs-keyword">void</span> main() {
  testWidgets(<span class="hljs-string">'test two_you_friend/pages/home_page/index.dart'</span>, (WidgetTester tester) <span class="hljs-keyword">async</span> {
     <span class="hljs-keyword">final</span> Widget testWidgets = HomePageIndex();
      <span class="hljs-keyword">await</span> tester.pumpWidget(
          <span class="hljs-keyword">new</span> MaterialApp(
              home: testWidgets
          )
      );
      expect(find.byWidget(testWidgets), findsOneWidget);
  });
}
</code></pre>
<p data-nodeid="39113">以上就完成了一个项目的初始化，比较简单的三个步骤。后期开发过程中可以使用 run 和 st 命令来提升研发效率。</p>
<h3 data-nodeid="39114">总结</h3>
<p data-nodeid="59049">以上就是本课时的主要内容，本课时通过工具化的方式来初始化项目，学完本课时你需要掌握 Flutter 项目基础结构，需要了解 flutter-pro-cli 的一个简单应用，最后希望你使用本课时的工具（或者手动的方式）创建一个 two you friend 项目，后面的课时我会逐步在该项目基础上完善功能。</p>

---

### 精选评论

##### **威：
> 老师，执行这个命令（flutter-pro-cli run check）报错：Error: spawn flutter ENOENT at Process.ChildProcess._handle.onexit (internal/child_process.js:190:19) at onErrorNT (internal/child_process.js:362:16) at _combinedTickCallback (internal/process/next_tick.js:139:11) at process._tickCallback (internal/process/next_tick.js:181:9)请问是什么原因？下面两个命令都能运行成功flutter -vflutter run lib/main.dart

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可能因为flutter版本更新有一些改变，你尝试在本地运行以下几个命令：dartfmt -w --fix lib/ 和 dartanalyzer lib 以及 flutter test ，如果有问题，在 github 上反馈下你的 flutter 版本，我更新一个脚本命令。

##### **斐：
> 'dartfmt' 不是内部或外部命令，也不是可运行的程序或批处理文件。'dartanalyzer' 不是内部或外部命令，也不是可运行的程序或批处理文件。老师，运行flutter-pro-cli run check时报这个错，什么原因亚

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你有增加环境变量吗，把 dartfmt 和 dartanalyzer ，你可以看下你环境变量的设置，如果没有加的话，你可以看下专栏的第 3 课时，里面有配置环境变量的方法。把这个flutter路径下的这个也加入到环境变量中去，{这里是flutter sdk路径}/bin/cache/dart-sdk/bin

##### *帆：
> mac 下报错Start check file formatnode:buffer:317 throw new ERR_INVALID_ARG_TYPE( ^TypeError [ERR_INVALID_ARG_TYPE]: The first argument must be of type string or an instance of Buffer, ArrayBuffer, or Array or an Array-like Object. Received an instance of Error at new NodeError (node:internal/errors:259:15) at Function.from (node:buffer:317:9) at iconvDecode (/usr/local/lib/node_modules/flutter-pro-cli/lib/check_file.js:39:32) at /usr/local/lib/node_modules/flutter-pro-cli/lib/check_file.js:19:33 at ChildProcess.exithandler (node:child_process:316:5) at ChildProcess.emit (node:events:327:20) at maybeClose (node:internal/child_process:1055:16) (node:internal/child_process:441:11) at Socket.emit (node:events:327:20) (node:net:655:12) { code: 'ERR_INVALID_ARG_TYPE'}

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个问题已经修复，如果还有问题，你直接在 github 上的 issue 给我留言，我会回复你。https://github.com/love-flutter/flutter-pro-cli/issues

##### **彬：
> 按照我们前面课时所设计的一些基础规范，这里需要两个基础的工具 dartfmt 和 dartanalyzer。将这两个工具整合在一起，一个为 shell 脚本和一个为 bat 脚本，整合后的文件叫作 format_check.sh 和 format_check.bat

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的哈，具体这里有什么问题呢？

##### **珠：
> 我的flutter地址已经加在系统变量中了，请问我脚手架报错spawn flutter ENORNT是为什么啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果已经加了环境变量，可以在当前运行的命令下，运行下 flutter -v，如果能运行成功，你再试下 flutter run lib/main.dart。如果这个命令没有成功，就检查下是否在当前项目目录运行该脚手架。

##### **宜：
> Windows 10下面运行‘flutter-pro-cli run check’报错Error: Command failed: ./format_check.bat'.' is not recognized as an internal or external command,operable program or batch file. '.' is not recognized as an internal or external command,operable program or batch file.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢提出问题，已经优化处理，麻烦再重试下。

##### Koenger：
> global文件放哪呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 全局共享状态的话，可以放在model中，在model创建一个全局状态文件夹。

