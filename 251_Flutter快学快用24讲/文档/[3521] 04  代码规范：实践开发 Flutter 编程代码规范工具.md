<p data-nodeid="25489" class="te-preview-highlight">在实践编程之前，我们先来掌握代码规范，毕竟优秀的编程代码从规范开始。</p>
<h3 data-nodeid="25490">命名规范</h3>
<p data-nodeid="25491">命名规范中包括了文件以及文件夹的命名规范，常量和变量的命名规范，类的命令规范。Dart 中只包含这三种命名标识。</p>
<ul data-nodeid="25492">
<li data-nodeid="25493">
<p data-nodeid="25494">AaBb 类规范，首字母大写驼峰命名法，例如 IsClassName，常用于类的命名。</p>
</li>
<li data-nodeid="25495">
<p data-nodeid="25496">aaBb 类规范，首字母小写驼峰命名法，例如 isParameterName，常用于常量以及变量命名。</p>
</li>
<li data-nodeid="25497">
<p data-nodeid="25498">aa_bb 类规范，小写字母下划线连接法，例如 is_a_flutter_file_name，常用于文件及文件夹命名。</p>
</li>
</ul>
<h3 data-nodeid="25499">注释规范</h3>
<p data-nodeid="25500">注释的目的是生成我们需要的文档，从而增强项目的可维护性。</p>
<h4 data-nodeid="25501">单行注释</h4>
<p data-nodeid="25502">单行注释主要是“ // ”这类标示的注释方法，这类注释与其他各类语言使用的规范一致。单行注释主要对于单行代码逻辑进行解释，为了避免过多注释，主要是在一些理解较为复杂的代码逻辑上进行注释。</p>
<p data-nodeid="25503">比如，下面这段代码没有注释，虽然你看上下文也会知道这里表示的是二元一次方程的 ∆ ，但是却不知道如果 ∆ 大于 0 ，为什么 x 会等于 2。</p>
<pre class="lang-dart" data-nodeid="25504"><code data-language="dart"><span class="hljs-keyword">if</span> ( b * b - <span class="hljs-number">4</span> * a * c &gt; <span class="hljs-number">0</span> ) {
  x = <span class="hljs-number">2</span>;
}
</code></pre>
<p data-nodeid="25505">如果加上注释则显得逻辑清晰容易理解，修改后如下所示。</p>
<pre class="lang-dart" data-nodeid="25506"><code data-language="dart"><span class="hljs-comment">// 当∆大于0则表示方程x个解，x则为2</span>
<span class="hljs-keyword">if</span> ( b * b - <span class="hljs-number">4</span> * a * c &gt; <span class="hljs-number">0</span> ) {
  x = <span class="hljs-number">2</span>;
}
</code></pre>
<p data-nodeid="25507">虽然单行注释大家都比较了解，但我这里还是多解释了下如何应用，主要是希望大家规范化使用，减少不必要的代码注释。</p>
<h4 data-nodeid="25508">多行注释</h4>
<p data-nodeid="25509">在 Dart 中由于历史原因（前后对多行注释方式进行了修改）有两种注释方式，一种是 /// ，另外一种则是 / **......* / 或者 /*......*/ ，这两种都可以使用。/**......*/ 和 /*......*/ 这种块级注释方式在其他语言（比如 JavaScript ）中是比较常用的，但是在 Dart 中我们更倾向于使用 /// ，后续我们所有的代码都按照这个规范来注释。</p>
<p data-nodeid="25510">多行注释涉及类的注释和函数的注释。两者在注释方法上一致。首先是用一句话来解释该类或者函数的作用，其次使用空行将注释和详细注释进行分离，在空行后进行详细的说明。如果是类，在详细注释中，补充该类作用，其次应该介绍返回出去的对象功能，或者该类的核心方法。如果是函数，则在详细注释中，补充函数中的参数以及返回的数据对象。</p>
<p data-nodeid="25511">假设有一个 App 首页的库文件，其中包含类 HomePage ， HomePage 中包含两个方法，一个是 getCurrentTime ，另一个是 build 方法，代码注释如下（未实现其他部分代码）。</p>
<pre class="lang-dart" data-nodeid="25512"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">APP 首页入口</span></span>
<span class="hljs-comment">/// 
<span class="markdown">/// 本模块函数，加载状态类组件HomePageState</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePage</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatefulWidget</span> </span>{
  <span class="hljs-meta">@override</span>
  createState() =&gt; <span class="hljs-keyword">new</span> HomePageState();
}
<span class="hljs-comment">/// <span class="markdown">首页有状态组件类</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 主要是获取当前时间，并动态展示当前时间</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageState</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">State</span>&lt;<span class="hljs-title">HomePage</span>&gt; </span>{
  <span class="hljs-comment">/// <span class="markdown">获取当前时间戳</span></span>
  <span class="hljs-comment">///
  <span class="markdown">/// [prefix]需要传入一个前缀信息</span></span>
  <span class="hljs-comment">/// <span class="markdown">返回一个字符串类型的前缀信息：时间戳</span></span>
  <span class="hljs-built_in">String</span> getCurrentTime(<span class="hljs-built_in">String</span> prefix) {
  }
  <span class="hljs-comment">/// <span class="markdown">有状态类返回组件信息</span></span>
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
  }
}
</code></pre>
<h4 data-nodeid="25513">注释文档生成</h4>
<p data-nodeid="25514">根据上面的代码注释内容，我们利用一个官方工具来将当前项目中的注释转化为文档。该工具的执行命令在 Dart 执行命令的同一个目录下，如果你在课时 03 中已经添加了 dart 命令行工具，那么该工具就可以直接使用了，如果没有则需要按照 03 课时中的方法，重新配置 dart 的运行命令的环境变量，这里主要演示下通过规范化的代码注释生成的文档。</p>
<p data-nodeid="25515">打开命令行工具进入当前项目，或者在 Android Studio 点击界面上的 Terminal 打开命令行窗口，运行如下命令。</p>
<pre class="lang-java" data-nodeid="25516"><code data-language="java">dartdoc
</code></pre>
<p data-nodeid="25517">运行结束后，会在当前项目目录生成一个 doc 的文件夹。在生成文件夹中，可以直接打开 doc/api/index.html 文件，你就会看到如图 1 所示的文档界面。</p>
<p data-nodeid="25518"><img src="https://s0.lgstatic.com/i/image/M00/22/66/Ciqc1F7sNjeAE5ykAAFKDSwqzfU381.png" alt="image (7).png" data-nodeid="25667"><br>
图 1 生成文档的整体界面结构</p>
<p data-nodeid="25519">接下来我们打开 HomePageState 类，可以看到如图 2 中的效果。</p>
<p data-nodeid="25520"><img src="https://s0.lgstatic.com/i/image/M00/22/72/CgqCHl7sNkGAMQRQAADbY3c-XN8695.png" alt="image (8).png" data-nodeid="25673"><br>
图 2 HomePageState 的注释文档</p>
<p data-nodeid="25521">其次再打开函数 getCurrentTime 可以看到图 3 的效果。从效果看，我们的文档已经生成了，而且效果很好。</p>
<p data-nodeid="25522"><img src="https://s0.lgstatic.com/i/image/M00/22/72/CgqCHl7sNkiAYZqRAAGMc2znMSs481.png" alt="image (9).png" data-nodeid="25679"><br>
图 3 getCurrentTime 的注释文档</p>
<p data-nodeid="25523">以上是使用标准的代码注释生成的文档，利用这种方式将大大提升项目的可维护性，希望大家在项目初期就要做好此类规范。</p>
<h3 data-nodeid="25524">库引入规范</h3>
<p data-nodeid="25525">Dart 为了保持代码的整洁，规范了 import 库的顺序。将 import 库分为了几个部分，每个部分使用空行分割。分为 dart 库、package 库和其他的未带协议头（例如下面中的 util.dart ）的库。其次相同部分按照模块的首字母的顺序来排列，例如下面的代码示例：</p>
<pre class="lang-dart" data-nodeid="25526"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'dart:developer'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/pages/home_page.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'util.dart'</span>;
</code></pre>
<h3 data-nodeid="25527">代码美化</h3>
<p data-nodeid="25528">在 Dart 中同样有和前端一样的工具 pritter ，在 Dart 中叫作 dartfmt ，该工具和 dartdoc 一样，已经包含在 Dart SDK 中，因此可以直接运行如下命令检查是否生效。</p>
<pre class="lang-java" data-nodeid="25529"><code data-language="java">dartfmt -h
</code></pre>
<p data-nodeid="25530">既然有此类工具，我们就来看下如何应用工具来规范和美化我们的代码结构。</p>
<h4 data-nodeid="25531">dartfmt</h4>
<p data-nodeid="25532">dartfmt 工具的规范包括了以下几点：</p>
<ul data-nodeid="25533">
<li data-nodeid="25534">
<p data-nodeid="25535">使用空格而不是 tab；</p>
</li>
<li data-nodeid="25536">
<p data-nodeid="25537">在一个完整的代码逻辑后面使用空行区分；</p>
</li>
<li data-nodeid="25538">
<p data-nodeid="25539">二元或者三元运算符之间使用空格；</p>
</li>
<li data-nodeid="25540">
<p data-nodeid="25541">在关键词 , 和 ; 之后使用空格；</p>
</li>
<li data-nodeid="25542">
<p data-nodeid="25543">一元运算符后请勿使用空格；</p>
</li>
<li data-nodeid="25544">
<p data-nodeid="25545">在流控制关键词，例如 for 和 while 后，使用空格区分；</p>
</li>
<li data-nodeid="25546">
<p data-nodeid="25547">在 ( [ { } ] ) 符号后请勿使用空格；</p>
</li>
<li data-nodeid="25548">
<p data-nodeid="25549">在 { 后前使用空格；</p>
</li>
<li data-nodeid="25550">
<p data-nodeid="25551">使用 . 操作符，从第二个 . 符号后每次都使用新的一行。</p>
</li>
</ul>
<p data-nodeid="25552">其他规范可以参考 <a href="https://github.com/dart-lang/dart_style/wiki/Formatting-Rules" data-nodeid="25706">dartfmt</a> 的官网。了解完以上规范后，我们现在将上面的 home_page.dart 进行修改，将部分代码修改为不按照上面规范的结构，代码修改如下：</p>
<pre class="lang-dart" data-nodeid="25553"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">APP 首页入口</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 本模块函数，加载状态类组件HomePageState</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePage</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatefulWidget</span></span>{
  <span class="hljs-meta">@override</span>
  createState() =&gt; <span class="hljs-keyword">new</span> HomePageState();
}
<span class="hljs-comment">/// <span class="markdown">首页有状态组件类</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 主要是获取当前时间，并动态展示当前时间</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageState</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">State</span>&lt;<span class="hljs-title">HomePage</span>&gt; </span>{
  <span class="hljs-comment">/// <span class="markdown">获取当前时间戳</span></span>
  <span class="hljs-comment">///
  <span class="markdown">/// [prefix]需要传入一个前缀信息</span></span>
  <span class="hljs-comment">/// <span class="markdown">返回一个字符串类型的前缀信息：时间戳</span></span>
  <span class="hljs-built_in">String</span> getCurrentTime( <span class="hljs-built_in">String</span> prefix ) {
  }
  <span class="hljs-comment">/// <span class="markdown">有状态类返回组件信息</span></span>
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
  }
}
</code></pre>
<p data-nodeid="25554">上面 getCurrentTime 的参数和 { 没有按照 dartfmt 规范来处理，在当前目录下打开 Terminal，然后先运行以下命令来修复当前的代码规范：</p>
<pre class="lang-java" data-nodeid="25555"><code data-language="java">&nbsp;dartfmt -w --fix lib/
</code></pre>
<p data-nodeid="25556">运行成功后，你将看到当前 home_page.dart 修改为如下代码：</p>
<pre class="lang-dart" data-nodeid="25557"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">APP 首页入口</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 本模块函数，加载状态类组件HomePageState</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePage</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatefulWidget</span> </span>{
  <span class="hljs-meta">@override</span>
  createState() =&gt; HomePageState();
}
<span class="hljs-comment">/// <span class="markdown">首页有状态组件类</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 主要是获取当前时间，并动态展示当前时间</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageState</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">State</span>&lt;<span class="hljs-title">HomePage</span>&gt; </span>{
  <span class="hljs-comment">/// <span class="markdown">获取当前时间戳</span></span>
  <span class="hljs-comment">///
  <span class="markdown">/// [prefix]需要传入一个前缀信息</span></span>
  <span class="hljs-comment">/// <span class="markdown">返回一个字符串类型的前缀信息：时间戳</span></span>
  <span class="hljs-built_in">String</span> getCurrentTime(<span class="hljs-built_in">String</span> prefix) {}
  <span class="hljs-comment">/// <span class="markdown">有状态类返回组件信息</span></span>
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {}
}
</code></pre>
<p data-nodeid="25558">可以看到两处不符合规范的被修复了，{ 前无空格问题，和 getCurrentTime 参数空格问题。</p>
<h3 data-nodeid="25559">工具化</h3>
<p data-nodeid="25560">上面介绍了这些规范，在 Dart 中同样存在和 eslint 一样的工具 dartanalyzer 来保证代码质量。</p>
<p data-nodeid="25561">该工具（ dartanalyzer ）已经集成在 Dart SDK ，你只需要在 Dart 项目根目录下新增analysis_options.yaml 文件，然后在文件中按照规范填写你需要执行的规则检查即可，目前现有的检查规则可以参考 <a href="https://dart-lang.github.io/linter/lints/" data-nodeid="25722">Dart linter rules</a> 规范。</p>
<p data-nodeid="25562">为了方便，我们可以使用现成已经配置好的规范模版，这里有两个库 <a href="https://s0pub0dev.icopy.site/packages/pedantic" data-nodeid="25727">pedantic</a> 和 <a href="https://s0dart0dev.icopy.site/guides/language/effective-dart" data-nodeid="25733">effective_dart</a> 可以参照使用。如果我们需要在项目中，使用它们两者之一，可以在项目配置文件（ pubspec.yaml ）中新增如下两行配置：</p>
<pre class="lang-dart" data-nodeid="25563"><code data-language="dart">dependencies:
  flutter:
    sdk: flutter
  pedantic: ^<span class="hljs-number">1.8</span><span class="hljs-number">.0</span>
  # The following adds the Cupertino Icons font to your application.
  # Use <span class="hljs-keyword">with</span> the CupertinoIcons <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">for</span> <span class="hljs-title">iOS</span> <span class="hljs-title">style</span> <span class="hljs-title">icons</span>.
  <span class="hljs-title">cupertino_icons</span>: ^0.1.2
<span class="hljs-title">dev_dependencies</span>:
  <span class="hljs-title">flutter_test</span>:
    <span class="hljs-title">sdk</span>: <span class="hljs-title">flutter</span>
  <span class="hljs-title">pedantic</span>: ^1.8.0
</span></code></pre>
<p data-nodeid="25564">配置完成以后，在当前项目路径下运行 flutter pub upgrade 。接下来在本地新增的 analysis_options.yaml 文件中新增如下配置：</p>
<pre class="lang-java" data-nodeid="25565"><code data-language="java">include: <span class="hljs-keyword">package</span>:pedantic/analysis_options.<span class="hljs-number">1.8</span>.<span class="hljs-number">0.</span>yaml
</code></pre>
<p data-nodeid="25566">如果我们认为 pedantic 不满足我们的要求，我们再根据 <a href="https://dart-lang.github.io/linter/lints/" data-nodeid="25741">Dart linter rules</a> 规范，前往选择自己需要的规范配置，修改下面的配置：</p>
<pre class="lang-dart" data-nodeid="25567"><code data-language="dart">include: package:pedantic/analysis_options<span class="hljs-number">.1</span><span class="hljs-number">.8</span><span class="hljs-number">.0</span>.yaml
analyzer:
  strong-mode:
    implicit-casts: <span class="hljs-keyword">false</span>
linter:
  rules:
    # STYLE
    - camel_case_types
    - camel_case_extensions
    - file_names
    - non_constant_identifier_names
    - constant_identifier_names # prefer
    - directives_ordering
    - lines_longer_than_80_chars # avoid
    # DOCUMENTATION
    - package_api_docs # prefer
    - public_member_api_docs # prefer
</code></pre>
<p data-nodeid="25568">我在 pedantic 的基础上又增加了一些对于样式和文档的规范，增加完成以上配置后，运行如下命令可进行检查。</p>
<pre class="lang-plain" data-nodeid="25569"><code data-language="plain">dartanalyzer lib
</code></pre>
<p data-nodeid="25570">运行完成以后，你可以看到一些提示、警告或者报错信息，具体提示如图 4 的问题：</p>
<p data-nodeid="25571"><img src="https://s0.lgstatic.com/i/image/M00/22/72/CgqCHl7sNqmAMpqEAAESy_g_9Ag796.png" alt="image (10).png" data-nodeid="25747"><br>
图 4 dartanalyzer 规则检查运行结果</p>
<p data-nodeid="25572">图 4 中的一些问题已经非常详细，包括以下几点：</p>
<ul data-nodeid="25573">
<li data-nodeid="25574">
<p data-nodeid="25575">没有为 main 类中的 public 方法增加文档说明；</p>
</li>
<li data-nodeid="25576">
<p data-nodeid="25577">在 main 类中 import 了developer 库，但是未使用；</p>
</li>
<li data-nodeid="25578">
<p data-nodeid="25579">在 main 类中 import 了 home_page.dart 库，但是未使用；</p>
</li>
<li data-nodeid="25580">
<p data-nodeid="25581">在 home_page.dart 中的 getCurrentTime 使用了 String 返回类型，但是未返回相应类型；</p>
</li>
<li data-nodeid="25582">
<p data-nodeid="25583">在 home_page.dart 中的 build 方法 使用了 Widaget 返回类型，但是未返回相应类型。</p>
</li>
</ul>
<p data-nodeid="25584">这些问题非常清晰地说明了我们目前代码存在的问题，有了以上工具化的校验检查，我们在做团队代码规范的时候，就非常简单。</p>
<h3 data-nodeid="25585">综合实践</h3>
<p data-nodeid="25586">学完本课时，我们按照以上的标准来实践一下。在上一课时中，我已经教大家怎么去实现一个比较简单的 Hello Flutter ，现在我希望实现一个显示当前时间的功能 APP 。</p>
<p data-nodeid="25587">以下是我的开发步骤，这里就涉及了上面所有的命名规范：</p>
<ol data-nodeid="25588">
<li data-nodeid="25589">
<p data-nodeid="25590">在 lib 下创建一个 pages 目录;</p>
</li>
<li data-nodeid="25591">
<p data-nodeid="25592">在 pages 下创建一个类为 home_page.dart 文件;</p>
</li>
<li data-nodeid="25593">
<p data-nodeid="25594">在 home_page.dart 文件中创建两个类，一个是 HomePage，另一个是 HomePageState；</p>
</li>
<li data-nodeid="25595">
<p data-nodeid="25596">在 HomePageState 类创建两个方法，一个是带返回 String 类型的 getCurrentTime 方法，另一个是带返回 Widget 类型的 build 方法（类似于 React 中的 render 方法）；</p>
</li>
<li data-nodeid="25597">
<p data-nodeid="25598">实现两个方法，具体可以查看以下代码；</p>
</li>
<li data-nodeid="25599">
<p data-nodeid="25600">在 main 函数中引入 home_page.dart 模块，并调用 HomePage 类。</p>
</li>
</ol>
<p data-nodeid="25601">具体 main.dart 和 home_page.dart 代码分别如下：</p>
<ul data-nodeid="25602">
<li data-nodeid="25603">
<p data-nodeid="25604">main.dart</p>
</li>
</ul>
<pre class="lang-dart" data-nodeid="25605"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/pages/home_page.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">APP 核心入口文件</span></span>
<span class="hljs-keyword">void</span> main() =&gt; runApp(MyApp());
<span class="hljs-comment">/// <span class="markdown">MyApp 核心入口界面</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyApp</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">// This widget is the root of your application.</span>
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> MaterialApp(
        title: <span class="hljs-string">'Two You'</span>,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: Scaffold(
            appBar: AppBar(
              title: Text(<span class="hljs-string">'Two You'</span>),
            ),
            body: Center(
              child: HomePage(),
            )));
  }
}
</code></pre>
<ul data-nodeid="25606">
<li data-nodeid="25607">
<p data-nodeid="25608">home_page.dart</p>
</li>
</ul>
<pre class="lang-dart" data-nodeid="25609"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:intl/intl.dart'</span>; <span class="hljs-comment">// 需要在pubspec.yaml增加该模块</span>

<span class="hljs-comment">/// <span class="markdown">APP 首页入口</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 本模块函数，加载状态类组件HomePageState</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePage</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatefulWidget</span> </span>{
  <span class="hljs-meta">@override</span>
  createState() =&gt; HomePageState();
}
<span class="hljs-comment">/// <span class="markdown">首页有状态组件类</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 主要是获取当前时间，并动态展示当前时间</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageState</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">State</span>&lt;<span class="hljs-title">HomePage</span>&gt; </span>{
  <span class="hljs-comment">/// <span class="markdown">获取当前时间戳</span></span>
  <span class="hljs-comment">///
  <span class="markdown">/// [prefix]需要传入一个前缀信息</span></span>
  <span class="hljs-comment">/// <span class="markdown">返回一个字符串类型的前缀信息：时间戳</span></span>
  <span class="hljs-built_in">String</span> getCurrentTime(<span class="hljs-built_in">String</span> prefix) {
    <span class="hljs-built_in">DateTime</span> now = <span class="hljs-built_in">DateTime</span>.now();
    <span class="hljs-keyword">var</span> formatter = DateFormat(<span class="hljs-string">'yy-mm-dd H:m:s'</span>);
    <span class="hljs-built_in">String</span> nowTime = formatter.format(now);
    <span class="hljs-keyword">return</span> <span class="hljs-string">'<span class="hljs-subst">$prefix</span> <span class="hljs-subst">$nowTime</span>'</span>;
  }
  <span class="hljs-comment">/// <span class="markdown">有状态类返回组件信息</span></span>
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> Text(
      getCurrentTime(<span class="hljs-string">'当前时间'</span>)
    );
  }
}
</code></pre>
<p data-nodeid="25610">然后我们再运行 dartfmt 来美化代码结构，其次运行 dartanalyzer 工具来校验是否按照规范进行开发。上面代码已经是标准规范，因此你不会发现任何问题，如果你自己开发过程中有问题，则按照提示进行修改即可。如果规范检查完成以后，都没有任何问题后，我们再运行当前程序，结果如图 5 所示的效果。</p>
<p data-nodeid="25611"><img src="https://s0.lgstatic.com/i/image/M00/22/67/Ciqc1F7sNr-AAZloAAFNQgKibbk184.png" alt="image (11).png" data-nodeid="25788"><br>
图 5 home_page 页面效果</p>
<h3 data-nodeid="25612">总结</h3>
<p data-nodeid="25613">本课时主要介绍了命名规范、注释规范以及文档生成、库引入规范、代码美化，最后利用 dartanalyzer 来进行工具化校验保证项目代码质量。学完本课时以后，你需要掌握这些基础规范，其次特别需要掌握 dartfmt 和 dartanalyzer 工具的使用。</p>
<p data-nodeid="25614">为了上面演示效果更佳，我们可以将时间变成自动更新的方式，这里就会涉及 05 课时的生命周期内容。具体实现效果以及原理，我会在接下来的 05 课时生命周期以及 06 课时有/无状态组件中详细说明。</p>
<p data-nodeid="25615"><a href="https://github.com/love-flutter/flutter-column" data-nodeid="25798">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### **1400：
> 运行dartdoc报错dartdoc failed: Invalid argument(s) (path): Must not be null ArgumentError.checkNotNull (dart:core/errors.dart:194:27) _Directory._checkNotNull (dart:io/directory_impl.dart:276:19)怎么解决？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以把flutter doctor的结果贴一下后续，主要是想看下当前的flutter版本，因为flutter的某些版本在dartdoc上确实存在一些bug。这个问题，看起来是flutter dartdoc的问题。

##### **勃：
> 集成了文档工具和代码规范工具真不错。

##### **杰：
> 规范确实很重要，好的规范能减少开发成本，降低维护成本

##### **誉：
> 说实话作者讲的东西挺实用的

##### *辉：
> <div>若执行 dartdoc 报如下错误 ❌</div><div><br></div>dartdoc failed: Top level package requires Flutter but FLUTTER_ROOT environment variable not set.<div><br></div><div>需要配置 FLUTTER_ROOT 环境变量</div><div>export FLUTTER_ROOT=你的 flutter 安装目录<br></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 检查下你的环境变量的设置。

##### **威：
> dartanalyzer lib是老师笔误，应是dart analyze lib

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 之前版本是dartanalyzer lib，最新版本已经修改为dart analyze，具体可以看下这个链接的说明：https://dart.dev/tools/dart-analyze，后续我将这块进行更新下。

##### *其：
> 有些内容跟最新的文档不太一样了比如dartanalyzer lib现在是执行这个flutter analyze还是要多看文档。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 嗯，是的哈，随着版本的更新，有些信息是会有所不同，不过这两者都是可以的，flutter analyze 则是可以直接执行，简化了原来配置环境变量的过程。

##### **的阿科：
> dartdoc 生成文档,但是使用报错dartfmt -w --fix lib/ 美化lib包下面的代码dartanalyzer lib 工具化校验保证代码质量，感觉引入包的方式不太智能，然后yaml也没有自动生成，要记的东西挺多的，比如库名字和库版本和yaml文件中填写的内容，或许有工具可以帮我达到，只是我现在不知道

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 确实没有像前端一样可以自动化生成和保存的方式。你说要记住库和库版本，这点倒是可以去 https://pub.flutter-io.cn/ 这里搜索，有点像 npm 包库，不过这里还是需要使用英文关键词，比如我们像搜索 location，获取地理位置的，还是能够找到的。

##### *磊：
> 老师，类中的变量应该用'///'还是'//'？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用三个斜杆

##### *磊：
> 前面需要插入多少空格？2个还是4个？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 主要看你们习惯，代码缩进 2 或者 4 个都可以，我建议 2 个。

##### **1400：
> D:\flutter\project\flutter_studydartdoc没有生成注释文档，只有空的文件夹

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 运行期间有没有报错信息，其次可以把flutter doctor的结果贴一下。如果是和下面同一个问题，可以看看换下最新版本的flutter试试。

##### *益：
> 老师，执行dartdoc 生成失败Generation failed: AnalysisException: Cannot compute LIBRARY_ELEMENT for E:\01-flutter\hello_word\lib\main.dartCaused by Unexpected exception while performing BuildCompilationUnitElementTask for source F:\Flutter_SDK\flutter\bin\cache\pkg\sky_engine\lib\ui\window.dart

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你看下你的dartdoc版本，或者flutter版本，在某个版本是存在这样的bug，你可以试着升级下最新版本的flutter sdk。

##### **5880：
> dartdoc 报错:Documenting my_app...dartdoc failed: Invalid argument(s): join(null, "bin", "cache", "dart-sdk"): part 0 was null, but part 1 was not.#0 _validateArgList (package:path/src/context.dart:1102:5)#1 Context.join (package:path/src/context.dart:242:5)#2 join (package:path/path.dart:265:13)#3 (package:dartdoc/src/dartdoc_options.dart:1586:16)#4 DartdocSyntheticOption._valueAtFromSynthetic (package:dartdoc/src/dartdoc_options.dart:797:54)#5 DartdocOptionArgSynth.valueAt (package:dartdoc/src/dartdoc_options.dart:756:12)#6 DartdocOptionContext.sdkDir (package:dartdoc/src/dartdoc_options.dart:1404:44)#7 PubPackageBuilder.sdk (package:dartdoc/src/model/package_builder.dart:81:60)#8 PubPackageBuilder.buildPackageGraph (package:dartdoc/src/model/package_builder.dart:67:7)#9 Dartdoc.generateDocsBase (package:dartdoc/dartdoc.dart:175:41)#10Dartdoc.generateDocs (package:dartdoc/dartdoc.dart:216:34)#11 (package:dartdoc/dartdoc.dart:485:15)#12_rootRun (dart:async/zone.dart:1190:13)#13_CustomZone.run (dart:async/zone.dart:1093:19)#14_runZoned (dart:async/zone.dart:1630:10)#15runZonedGuarded (dart:async/zone.dart:1618:12)#16Dartdoc.executeGuarded (package:dartdoc/dartdoc.dart:483:5)#17main (file:///b/s/w/ir/cache/builder/src/third_party/dart/third_party/pkg/dartdoc/bin/dartdoc.dart:24:11)#18 (dart:isolate-patch/isolate_patch.dart:299:32)#19_RawReceivePortImpl._handleMessage (dart:isolate-patch/isolate_patch.dart:168:12)是为什么?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你看下flutternsdl版本，有个版本的确有这个问题，用最新的一个版本再尝试下，新版本是已经修复了这个问题的。

##### **生：
> 老师，麻烦问下dartanalyzer这个也是只能执行命令校验吗？有没有类似eslint自动检测的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一般 IDE 可以自带自动化检测，可以参考这个文档。https://flutter.cn/docs/get-started/editor

##### **豪：
> 生成的index.html点击对应的类加载失败

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这部分可能需要你提供更多的信息才知道，首先检查下在文档生成过程中是否报错，特别是你提到的失败的这个类。

##### **云：
> command not found: dartdoc怎么解决 还需要单独下载dartSDK吗？但是我的工程都能跑起来了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果你是mac，在命令行窗口运行下ls $PATH，看下有没有/dart-sdk/bin目录，如果没有根据我们第3课时的方法去配置下。如果是windows，直接去打开环境变量确认下是否有dart-sdk/bin路径。

##### **锋：
> dartfmt 和 dartanalyzer 这两个工具确实很实用！照着这个例子敲了遍代码，发现Android Studio里写代码没有XCode联想好，得区分大小写，实在不方便

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 把dartdoc也用上

##### **9612：
> analysis_options.yaml没有生成是怎么回事，配置如下dependencies:  flutter:    sdk: flutter  # The following adds the Cupertino Icons font to your application.  # Use with the CupertinoIcons class for iOS style icons.  cupertino_icons: ^0.1.2  provider: ^3.0.0  pedantic: ^1.8.0dev_dependencies:  flutter_test:    sdk: flutter  pedantic: ^1.8.0

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 这个需要自己创建一个就行了。

##### **冬：
> 执行dartdoc创建出的doc/api文件夹下没文档什么情况？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你确认下是否在当前项目目录，具体可能得结合你实际场景来看。

##### *欣：
> 很有帮助，刚开始接触flutter，提前掌握好代码规范，团队开发更高效。

