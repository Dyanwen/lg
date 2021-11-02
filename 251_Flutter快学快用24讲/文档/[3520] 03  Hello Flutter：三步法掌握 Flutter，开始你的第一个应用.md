<p data-nodeid="1285" class="">本课时将进入 Flutter 开发实践应用。在进入实践应用之前，我先讲解最基础的环境搭建，然后会应用 Dart 语言开发第一个 App  —  Hello Flutter，最后再讲解一些开发过程中常用的调试方法和工具。</p>


<p data-nodeid="3" class="">本课时需要一定的实践动手能力，因此在学习的时候建议你打开电脑按照里面的步骤进行学习。</p>
<h3 data-nodeid="4">第一步：环境搭建</h3>
<p data-nodeid="2139" class="">环境构建方法在官网已提供了非常详细的指引，你可以参考官网指引<a href="https://flutterchina.club/get-started/install/" data-nodeid="2143">《起步:安装 Flutter》</a>。这里我先介绍一些共性的问题，然后再分别从  Mac 系统 和 Windows 系统介绍其中比较有代表性的问题。</p>

<h4 data-nodeid="6">常见问题</h4>
<p data-nodeid="7">以下是大家很容易忽视的几个问题。</p>
<ul data-nodeid="2985">
<li data-nodeid="2986">
<p data-nodeid="2987"><strong data-nodeid="2999">环境要求</strong>，你需要注意 Flutter 的环境要求，很多人都会忽视这一点，导致在安装过程中遇到问题才会回头看环境要求，所以无论自己对配置如何了解，都需要按照官网的指引去检查每个配置项。</p>
</li>
<li data-nodeid="2988">
<p data-nodeid="2989"><strong data-nodeid="3004">Flutter 下载</strong>，请尽量下载当前稳定版本，避免因为不稳定版本导致的其他环境要求，导致安装不成功。</p>
</li>
<li data-nodeid="2990">
<p data-nodeid="2991"><strong data-nodeid="3013">Android Studio 工具安装</strong>，Flutter 的配置运行需要依赖 Android Studio 来完成，因此在安装之前可以先准备好 Android Studio 的安装配置，并且需要了解其中关于 Flutter 插件和 Dart 插件的安装，这些在 <a href="https://flutterchina.club/get-started/install/" data-nodeid="3011">Flutter 官网</a>有详细的解释说明。</p>
</li>
<li data-nodeid="2992">
<p data-nodeid="2993" class=""><strong data-nodeid="3018">Anroid Studio 出现 unable to access android sdk add-on list</strong>，出现这个问题，可以修改 Android Studio 安装目录 bin 下的 idea.properties 文件，在文件最后一行增加如下配置。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="4283"><code data-language="java">disable.android.first.run = <span class="hljs-keyword">true</span>
</code></pre>




<ul data-nodeid="7739">
<li data-nodeid="7740">
<p data-nodeid="7741"><strong data-nodeid="7751">Android Studio 网络代理</strong>，如果你的网络有代理，也需要进行配置，如果没有正确配置，将导致 Andorid Studio 提示 flutter pub upgrade 无法正常更新。</p>
</li>
<li data-nodeid="7742">
<p data-nodeid="7743"><strong data-nodeid="7762">Flutter <strong data-nodeid="7761"><strong data-nodeid="7760">D</strong></strong>octor 核心点检查</strong>，需要认真检查其中的每一项，对于其中的问题项，Doctor 一般会提供具体的解决方案。</p>
</li>
<li data-nodeid="7744">
<p data-nodeid="7745" class=""><strong data-nodeid="7767">点击 Finish 长久未响应</strong>（或者执行 flutter pub upgrade 未响应），这种情况会出现“This is taking an unexpectedly long time”提示，如果出现这个提示，很大可能是你的镜像配置没有按要求配置。你可以参考以下这段配置，第一个是 Flutter 的命令行工具，第二个则是 Dart 的命令行工具，后面两个镜像配置很关键。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="8602"><code data-language="java">PATH=$PATH:/Users/用户名/Downloads/flutter-main/bin
PATH=$PATH:/Users/用户名/Downloads/flutter-main/bin/cache/dart-sdk/bin
PUB_HOSTED_URL=https:<span class="hljs-comment">//pub.flutter-io.cn</span>
FLUTTER_STORAGE_BASE_URL=https:<span class="hljs-comment">//storage.flutter-io.cn</span>
</code></pre>









<ul data-nodeid="9437">
<li data-nodeid="9438">
<p data-nodeid="9439" class=""><strong data-nodeid="9445">Flutter SDK path not given</strong>，如果在创建 Flutter 项目时候提示“ Flutter SDK path not given“，则点击 Flutter SDK path 路径，然后选择我们前面安装的 Flutter SDK 路径即可。</p>
</li>
</ul>
<h4 data-nodeid="9440">Mac 系统上注意的点</h4>


<p data-nodeid="30">Mac 上的安装，我这里主要说明 Xcode 和 Mac 下的环境变量配置。</p>
<ul data-nodeid="31">
<li data-nodeid="32">
<p data-nodeid="33">Xcode 要升级到指定版本以上，由于 Flutter 需要应用 iOS 模拟器，因此对 Xcode 版本有一定要求。</p>
</li>
<li data-nodeid="34">
<p data-nodeid="35">Mac 下设置环境变量，其中涉及一些环境变量的配置，虽然网上有很多方法，官网也有提供，但我推荐大家使用如下方法，永久设置。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="12794"><code data-language="java">sudo vim ~/.bash_profile
</code></pre>
<p data-nodeid="14496">配置添加 Flutter 的安装路径，一般情况下会安装在你解压后运行的路径下。例如，下面我自己安装后的路径，安装完成后确定具体路径，然后在 bash_profile 文件中增加这行配置即可。</p>
<pre data-nodeid="16188" class=""><code>PATH=$PATH:/Users/用户名/Downloads/flutter-main/bin
</code></pre>
<p data-nodeid="16189">最后再运行加载，并运行测试。</p>










<pre class="lang-java" data-nodeid="17830"><code data-language="java">source ~/.bash_profile
flutter -h
</code></pre>
<h4 data-nodeid="17831">Windows 系统上注意的点</h4>




<p data-nodeid="40">Widows 系统安装需注意以下几点。</p>
<ul data-nodeid="41">
<li data-nodeid="42">
<p data-nodeid="43" class="">环境变量的设置，如果在 cmd 下没有 export 命令，前往系统属性下 -&gt; 环境变量，然后新建，按照变量名为  PUB_HOSTED_URL ，变量值为 <a href="https://pub.flutter-io.cn" data-nodeid="249">https://pub.flutter-io.cn</a> ，以及变量名为 FLUTTER_STORAGE_BASE_URL ，变量值为 <a href="https://storage.flutter-io.cn" data-nodeid="259">https://storage.flutter-io.cn</a> 进行配置，对应到官方文档如下配置。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="19103"><code data-language="java">export PUB_HOSTED_URL=https:<span class="hljs-comment">//pub.flutter-io.cn</span>
export FLUTTER_STORAGE_BASE_URL=https:<span class="hljs-comment">//storage.flutter-io.cn</span>
</code></pre>


<ul data-nodeid="45">
<li data-nodeid="46">
<p data-nodeid="47">配置 Flutter 运行环境，下载完成 Flutter SDK ，并放到指定的 C:\src\ 下，然后再次配置环境变量，需要在环境变量名为 PATH 的字段后面增加分号分割，并在分号后增加如下路径。</p>
</li>
</ul>
<pre class="lang-plain" data-nodeid="48"><code data-language="plain">C:\src\flutter\bin
</code></pre>
<ul data-nodeid="49">
<li data-nodeid="50">
<p data-nodeid="51">如果出现安装 Android SDK 时无法勾选 SDK ，需要重新卸载安装。这里需注意，在卸载时需勾选删除当前用户本地 Android Studio 配置，然后重新安装时，选择非 Program Files 目录。</p>
</li>
</ul>
<h3 data-nodeid="52">第二步：创建项目运行</h3>
<p data-nodeid="53">上面的配置安装完成后，我们就开始创建 Flutter 项目，这里我介绍的是 Android Studio IDE 的过程。</p>
<ol data-nodeid="20788">
<li data-nodeid="20789">
<p data-nodeid="20790">选择新建一个 Start a new Flutter Project ，然后选择 Flutter Application ，如图 1。</p>
</li>
</ol>
<p data-nodeid="21642" class=""><img src="https://s0.lgstatic.com/i/image/M00/21/12/Ciqc1F7pvnuANcQpAACTMSsFoo0714.png" alt="image" data-nodeid="21645"><br>
图 1 New Flutter Project</p>




<ol start="2" data-nodeid="26351">
<li data-nodeid="26352">
<p data-nodeid="26353">然后依次填写相应的 Project name 、Flutter SDK Path（如果配置好了会默认填写上，如果没有可以去重新选择）、Project location （具体的项目保存地址）、Descrition ，填写完成后，点击下一步，然后点击 finish 即可。</p>
</li>
<li data-nodeid="26354">
<p data-nodeid="26355">如果卡在 finish 这个环节，请强制退出，然后再重新打开，检查配置。具体解决办法可参考共性问题中的“点击 finish 长久未响应”问题。</p>
</li>
<li data-nodeid="26356" class="">
<p data-nodeid="26357">创建完成后，会看到如图 2 的项目目录结构。</p>
</li>
</ol>



<p data-nodeid="24207" class=""><img src="https://s0.lgstatic.com/i/image/M00/21/12/Ciqc1F7pvouAKi3mAAC2vjxyHVc774.png" alt="image" data-nodeid="24213"><br>
图 2 Flutter 项目目录结构</p>






<ol start="5" data-nodeid="28479">
<li data-nodeid="28480">
<p data-nodeid="28481">成功创建后，我们选择一个模拟器，然后在运行入口文件选择 main.dart ，最后点击右侧启动按钮进行编译运行。<strong data-nodeid="28488">如果下拉没有模拟器，Android Studio 会提供指引前往配置</strong>。</p>
</li>
</ol>
<p data-nodeid="28482" class=""><img src="https://s0.lgstatic.com/i/image/M00/21/1D/CgqCHl7pvrmAKD3eAAAhuRZycV0676.png" alt="image" data-nodeid="28491"><br>
图 3 运行启动说明</p>





<ol start="6" data-nodeid="31030">
<li data-nodeid="31031">
<p data-nodeid="31032">运行成功后，将会打开 iPhone 11 模拟器，然后启动我们的应用，如图 3。</p>
</li>
</ol>
<p data-nodeid="31033" class=""><img src="https://s0.lgstatic.com/i/image/M00/21/12/Ciqc1F7pvsSAKTXCAAGu5cF8GWk440.png" alt="image" data-nodeid="31037"><br>
图 4 iPhone 11 模拟器</p>





<p data-nodeid="78">以上就成功配置了 Flutter 运行环境和开发工具。</p>
<h3 data-nodeid="79">第三步：实现 Hello Flutter APP</h3>
<p data-nodeid="80">在实现一些编程之前，我先详细介绍工程目录中每个目录的作用，其次介绍如何进行修改代码，实现界面显示 Hello Flutter，最后再介绍三个常见的调试方法。</p>
<h4 data-nodeid="81">目录说明</h4>
<p data-nodeid="34406">上述图 2 中已有相关工程目录的截图，我现在分别介绍下每个目录的作用。</p>
<p data-nodeid="34407" class=""><img src="https://s0.lgstatic.com/i/image/M00/21/1E/CgqCHl7pvuqAfnGbAAC2vjxyHVc400.png" alt="image" data-nodeid="34411"><br>
图 2 Flutter 项目目录结构</p>






<ul data-nodeid="85">
<li data-nodeid="86">
<p data-nodeid="87"><strong data-nodeid="308">.idea</strong></p>
</li>
</ul>
<p data-nodeid="88">这个和 Flutter 无关，这里面主要是保留代码的修改历史。</p>
<ul data-nodeid="89">
<li data-nodeid="90">
<p data-nodeid="91"><strong data-nodeid="313">android</strong></p>
</li>
</ul>
<p data-nodeid="92">这个目录主要是和 Android 原生平台交互的工程代码，其目录结构和原生的 Android 项目基本一致，但是一些配置和代码结构是不同的。</p>
<ul data-nodeid="93">
<li data-nodeid="94">
<p data-nodeid="95"><strong data-nodeid="318">ios</strong></p>
</li>
</ul>
<p data-nodeid="96">这个目录主要也是和 iOS 原生平台交互的代码。</p>
<ul data-nodeid="97">
<li data-nodeid="98">
<p data-nodeid="99"><strong data-nodeid="323">lib</strong></p>
</li>
</ul>
<p data-nodeid="100">这个目录下的文件为 Flutter 项目核心代码，其中包含了一个 main.dart 入口文件。</p>
<ul data-nodeid="101">
<li data-nodeid="102">
<p data-nodeid="103"><strong data-nodeid="328">test</strong></p>
</li>
</ul>
<p data-nodeid="104">这个目录下的文件存放 Flutter 项目相关的测试文件。</p>
<ul data-nodeid="105">
<li data-nodeid="106">
<p data-nodeid="107"><strong data-nodeid="333">pubspec.yaml</strong></p>
</li>
</ul>
<p data-nodeid="108">该文件为 Flutter 项目配置文件，包括了项目名、项目描述、版本、运行环境以及开发和正式环境的第三方库，该文件与我们熟悉的 package.json 作用是类似的。</p>
<ul data-nodeid="109">
<li data-nodeid="110">
<p data-nodeid="111"><strong data-nodeid="338">pubspec.lock</strong></p>
</li>
</ul>
<p data-nodeid="112">这是自动生成的文件，里面指明了 pubspec.yaml 等依赖包和项目依赖库的具体版本号，该文件的功能和我们常见的 package.lock.json 作用类似。</p>
<ul data-nodeid="113">
<li data-nodeid="114">
<p data-nodeid="115"><strong data-nodeid="343">.metadata</strong></p>
</li>
</ul>
<p data-nodeid="116">这是自动生成的文件，里面记录了项目的属性信息。用于切换分支、升级 SDK 使用。</p>
<ul data-nodeid="117">
<li data-nodeid="118">
<p data-nodeid="119"><strong data-nodeid="348">.packages</strong></p>
</li>
</ul>
<p data-nodeid="120">这里面放置了项目依赖的库，对应在本机电脑上的绝对路径，为自动生成文件。如果项目出错或者无法找到某个库，可以把这个文件删除，重新自动配置即可。</p>
<p data-nodeid="121">.gitignore、README.md 与前端项目中的文件作用是一致的，这里就不详加说明。</p>
<p data-nodeid="122"><strong data-nodeid="372">在开发过程中我们只需要关注三个核心部分，代码开发<strong data-nodeid="367"><strong data-nodeid="366">放在</strong></strong> lib 下，test 存放我们的测试文件，项目配置文件<strong data-nodeid="369"><strong data-nodeid="368">放在</strong></strong> pubspec.yaml <strong data-nodeid="371"><strong data-nodeid="370">下</strong></strong>。</strong></p>
<h4 data-nodeid="123">Hello Flutter</h4>
<p data-nodeid="124">分析清楚文件目录后，在 lib 下修改 main.dart ，在该模块中打印 Hello Flutter 实现第一个 Flutter 应用开发。</p>
<ol data-nodeid="125">
<li data-nodeid="126">
<p data-nodeid="127">打开 main.dart ，将文件中 MaterialApp 下的 title 名字修改为 “Two You” ，将 home 下的 title 修改为 “Two You”，相关代码如下所示。</p>
</li>
</ol>
<pre class="lang-dart" data-nodeid="128"><code data-language="dart"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyApp</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">// This widget is the root of your application.</span>
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> MaterialApp(
      title: <span class="hljs-string">'Two You'</span>, <span class="hljs-comment">// app 的title信息 </span>
        primarySwatch: Colors.blue, <span class="hljs-comment">// 页面的主题颜色</span>
      ),
      home: MyHomePage(title: <span class="hljs-string">'Two You'</span>), <span class="hljs-comment">// 当前页面的 title 信息</span>
    );
  }
}
</code></pre>
<ol start="2" data-nodeid="37804">
<li data-nodeid="37805">
<p data-nodeid="37806">将 main.dart 中 Scaffold 下的 body 下的 children 下的第一个 Text 内容修改为 “Hello Flutter”，并去掉下面一个 Text，如下图 5。</p>
</li>
</ol>
<p data-nodeid="37807" class=""><img src="https://s0.lgstatic.com/i/image/M00/21/1E/CgqCHl7pvyyAAoyEAAGmjIkimiw325.png" alt="image" data-nodeid="37811"><br>
图 5 修改 main.dart 文件的代码指引</p>







<p data-nodeid="39500">修改完成后，保存文件，然后按照本课时中的”第二步：创建项目运行“运行本程序即可（如果已经运行过，保存文件模拟器会热加载），你将看到如下的结果，如图 6 所示。</p>
<p data-nodeid="39501" class=""><img src="https://s0.lgstatic.com/i/image/M00/21/1E/CgqCHl7pvzyAHXDxAAGn0n0MOsU471.png" alt="image" data-nodeid="39505"><br>
图 6 Hello Flutter 运行结果</p>




<p data-nodeid="137">上面的代码是基于最开始的 main.dart 进行，如果觉得修改原文件比较麻烦，我们可以简化为如下的代码：</p>
<pre class="lang-dart" data-nodeid="138"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">void</span> main() =&gt; runApp(MyApp());
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyApp</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">// This widget is the root of your application.</span>
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> MaterialApp(
      title: <span class="hljs-string">'Two You'</span>, <span class="hljs-comment">// app 的title信息&nbsp;</span>
      theme: ThemeData(
        primarySwatch: Colors.blue, <span class="hljs-comment">// 页面的主题颜色</span>
      ),
      home: Scaffold(
          appBar: AppBar(
            title: Text(<span class="hljs-string">'Two You'</span>), <span class="hljs-comment">// 当前页面的 title 信息</span>
          ),
          body:  Center(
            child: Text(<span class="hljs-string">'Hello Flutter'</span>), <span class="hljs-comment">// 当前页面的显示的文本信息</span>
          )
      )
    );
  }
}
</code></pre>
<h4 data-nodeid="139">调试方法</h4>
<p data-nodeid="140">代码运行调试在各种语言中都是比较基本的知识点，在 Flutter 中也应该掌握，这里我只介绍 Flutter 不同于其他语言的调试方法，包含以下几类：</p>
<ul data-nodeid="141">
<li data-nodeid="142">
<p data-nodeid="143"><strong data-nodeid="392">断点调试</strong></p>
</li>
</ul>
<p data-nodeid="144">这个知识点和大家熟悉的 Chrome 的断点调试基本一致，核心是在断点处查看当前各个数据的状态情况，但是需要使用 debug 模式运行。</p>
<ul data-nodeid="145">
<li data-nodeid="146">
<p data-nodeid="147"><strong data-nodeid="397">debugger 调试</strong></p>
</li>
</ul>
<p data-nodeid="148">在代码中增加一个断点语法，可以通过条件式的判断来进行断点，同样需要使用 debug 模式运行。</p>
<ul data-nodeid="149">
<li data-nodeid="150">
<p data-nodeid="151"><strong data-nodeid="402">界面调试</strong></p>
</li>
</ul>
<p data-nodeid="152">为了能够掌握具体的布局问题，在 Web 端，我们可以通过 Chrome 工具进行分析。虽然在 Flutter 中是没有 Chrome 工具，但是 Flutter 提供了可视化的界面调试方法。</p>
<p data-nodeid="153">上面提到的三点，其实在 Flutter 中提供了一个非常不错的工具。如果你是在 Android Studio 中的话，你可以直接点击下图 7 的按钮，将为你下载相应的组件，然后打开图 8 的界面调试框。如果你使用的是非 Android Studio ，可以使用命令行的方式，参考<a href="https://flutter.cn/docs/development/tools/devtools/cli" data-nodeid="407">官网</a>方式，首先安装 devtools 工具。</p>
<pre class="lang-java" data-nodeid="40778"><code data-language="java">pub global activate devtools
</code></pre>


<p data-nodeid="155">安装完成后，运行以下命令启动运行。</p>
<pre class="lang-java" data-nodeid="43312"><code data-language="java">pub global run devtools
</code></pre>
<p data-nodeid="45005"><img src="https://s0.lgstatic.com/i/image/M00/21/12/Ciqc1F7pv1iADPaQAABwzl3Sgow148.png" alt="image" data-nodeid="45009"><br>
图 7 Flutter 调试工具按钮指引</p>
<p data-nodeid="45006" class=""><img src="https://s0.lgstatic.com/i/image/M00/21/12/Ciqc1F7pv2CAVgmvAAMQ0qCy2Nw964.png" alt="image" data-nodeid="45014"><br>
图 8 Dart DevTools 工具</p>









<p data-nodeid="161">该套工具的详细介绍可以参考<a href="https://flutter.cn/docs/development/tools/devtools" data-nodeid="421">开发者工具</a>。</p>
<h3 data-nodeid="162">总结</h3>
<p data-nodeid="163">本课时介绍了如何三步开启第一个应用程序 Hello Flutter，包括环境搭建、创建项目以及运行、修改示例代码。学完本课时，你需要掌握环境搭建的方法以及如何创建运行项目。</p>

<p data-nodeid="165"><a href="https://github.com/love-flutter/flutter-column" data-nodeid="428">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### *磊：
> Running Gradle task 'assembleDebug'...FAILURE: Build failed with an exception.* What went wrong:Could not receive a message from the daemon.* Try:Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.* Get more help at https://help.gradle.orgException: Gradle task assembleDebug failed with exit code 1老师，我卡在这儿了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; flutter doctor -v 看下配置信息，如果配置正常，然后运行 flutter clean ，运行完成后再运行看看。

##### **铭：
> 遇到问题不要慌，最终还是成功了，却不确信因为哪个环节。没设置镜像，没关防火墙，没升级Gradle。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 为你点赞

##### **吟：
> 用Android Studio开发，如何配置的iphone 11模拟器，能说下安装过程吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你先在命令行运行 flutter doctor，会检查 xcode版本信息，如果没有问题。然后你再运行下这个链接下的“配置 iOS 运行环境部分”https://flutter.cn/docs/get-started/install/macos

##### **8082：
> 接上条 at org.gradle.wrapper.Install$1.call(Install.java:48) at org.gradle.wrapper.ExclusiveFileAccessManager.access(ExclusiveFileAccessManager.java:65) at org.gradle.wrapper.Install.createDist(Install.java:48) at org.gradle.wrapper.WrapperExecutor.execute(WrapperExecutor.java:128) at org.gradle.wrapper.GradleWrapperMain.main(GradleWrapperMain.java:61)Running Gradle task 'assembleDebug'...Running Gradle task 'assembleDebug'... Done  3.9sException: Gradle task assembleDebug failed with exit code 1看样子是jdk和gradle的锅，但却不知道怎么改，请教老师！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你这样，打开项目目录下的 android/gradle/wrapper/graddle-wrapper.properties文件，然后看下里面的distributionUrl，先看下是否为https的，如果不是修改下。如果是你在命令中 curl 一下，如果也 curl 不通，有可能是你代理的问题。

##### **泽：
> 不是应该配置flutter的bin目录吗 配置安装路径是不是 把dartsdk也放path了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的哈，都是设置bin目录

