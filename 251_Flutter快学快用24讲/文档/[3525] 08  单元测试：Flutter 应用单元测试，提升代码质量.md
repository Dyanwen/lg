<p data-nodeid="669" class="">之前已经讲解了 Flutter 所有基础的知识点，本课时介绍如何保证组件代码的质量，以此来确保我们在代码开发过程中或者在重构过程中的代码质量。</p>
<h3 data-nodeid="670">单元测试</h3>
<p data-nodeid="671">单元测试的概念是针对程序中最小单位来进行校验的工作，在 Flutter 中最小的单位是组件。由于我们扩展了一些模块比如 Model（Provider）、Struct（数据结构部分），因此这里也需要介绍下这两部分的单元测试。</p>
<h4 data-nodeid="672">目录结构</h4>
<p data-nodeid="673">为了保持一致性，我们在 test 单元测试目录，创建与项目结构目录一致的结构，如图 1 所示。</p>
<p data-nodeid="674"><img src="https://s0.lgstatic.com/i/image/M00/2B/C9/Ciqc1F7_AHGAJ8__AABUPoaGi10666.png" alt="image (9).png" data-nodeid="735"><br>
图 1 单元测试目录结构</p>
<p data-nodeid="675">单元测试目录结构下的测试文件命名也是按照原组件命名方式，但是需要在组件命名后面增加 test 后缀。例如，我们需要对 article_comments.dart 文件进行单元测试，根据规则将其命名为 article_comments_test.dart。</p>
<h4 data-nodeid="676">前期准备</h4>
<p data-nodeid="677">首先我们需要在 pubspec.yaml 中增加相应的 flutter_test 第三库，一般项目初始化后，会自动在 dev_dependencies 中引入，最后执行 flutter pub get 更新本地第三库即可。目录结构目前还是需要手动创建，在下一课时，我会在脚手架中自动化创建。</p>
<h3 data-nodeid="678">Struct 的单元测试</h3>
<p data-nodeid="679">Struct 的目的是保证数据结构的安全，避免因为动态数据结构而引发客户端的 Crash 问题，因此做好数据结构的单元测试非常必要。Struct 的结构比较简单，只有一个构造函数，在构造函数中存在必须和可选参数，单元测试部分主要是验证这个构造函数即可。</p>
<p data-nodeid="680">在上一课时中，我们创建了三个 Struct ，这里着重介绍较为复杂的 comment_info_struct.dart 的测试用例写法，代码如下。</p>
<pre class="lang-dart" data-nodeid="681"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter_test/flutter_test.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/util/struct/comment_info_struct.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/util/struct/user_info_struct.dart'</span>;
<span class="hljs-keyword">void</span> main() {
  <span class="hljs-keyword">final</span> UserInfoStruct userInfo = UserInfoStruct(<span class="hljs-string">'test'</span>, <span class="hljs-string">'http://test.com'</span>);
  test(<span class="hljs-string">'test-userinfo'</span>, () {
    <span class="hljs-keyword">final</span> CommentInfoStruct commentInfo =
      CommentInfoStruct(userInfo, <span class="hljs-string">'comment test'</span>);
    expect(commentInfo.comment == <span class="hljs-string">'comment test'</span>, <span class="hljs-keyword">true</span>);
    expect(commentInfo.userInfo.nickname == <span class="hljs-string">'test'</span>, <span class="hljs-keyword">true</span>);
    expect(commentInfo.userInfo.headerImage, <span class="hljs-string">'http://test.com'</span>);
  });
}
</code></pre>
<p data-nodeid="682">第 1 行代码引入 flutter_test 第三方库，第 3 和 4 行引入本次测试需要的 struct 结构库。测试文件的所有测试逻辑都在 main 函数中。在第 7 行中使用 UserInfoStruct 创建 userInfo ，Flutter 中的类以及库测试都是以 test 函数为测试方法，test 包含两个参数，一个是测试的描述，另外一个是测试的核心逻辑。</p>
<p data-nodeid="683">测试的核心逻辑中有一个 expect 方法，该方法可以在代码前使用一个条件判断语句，例如等于、大于、小于等等，而第二个参数可以是任何数据。如果 expect 的前后两个值相等，则测试用例通过，如果不相等则不通过。</p>
<p data-nodeid="684">代码完成以后，我们在根目录执行下面的命令。</p>
<pre class="lang-plain" data-nodeid="685"><code data-language="plain">flutter test
</code></pre>
<p data-nodeid="686">执行完成后，就可以看到以下结果，这表明测试用例已全部通过。</p>
<pre class="lang-plain" data-nodeid="687"><code data-language="plain">00:04 +1: All tests passed!
</code></pre>
<h3 data-nodeid="688">Model 的单元测试</h3>
<p data-nodeid="689">Model 的测试和 Struct 基本一样，不过在 Model 中有较多方法，因此需要增加一些类方法的测试。这里我们使用 like_num_model.dart 作为测试文件，在 test 目录下的 model 文件夹中新增测试文件 like_num_model_test.dart ，并在实现如下测试代码。</p>
<pre class="lang-dart" data-nodeid="690"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter_test/flutter_test.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/model/like_num_model.dart'</span>;
<span class="hljs-keyword">void</span> main() {
  <span class="hljs-keyword">final</span> LikeNumModel likeNumModel = LikeNumModel();
  test(<span class="hljs-string">'test like model value'</span>, () {
    expect(likeNumModel.value, <span class="hljs-number">0</span>);
  });
  test(<span class="hljs-string">'test like model like method'</span>, () {
    likeNumModel.like();
    expect(likeNumModel.value, <span class="hljs-number">1</span>);
    likeNumModel.like();
    expect(likeNumModel.value, <span class="hljs-number">2</span>);
  });
}
</code></pre>
<p data-nodeid="691">代码中第 1 行和第 3 行都是引入相应的库以及测试库文件，其次以 main 为测试入口，在 main 中调用 LikeNumModel 初始化并获得操作句柄，然后分为两部分，一部分测试状态属性，另一部分测试相应状态属性变更的类方法。</p>
<h3 data-nodeid="692">组件的单元测试</h3>
<p data-nodeid="693">上面两部分测试代码逻辑较为简单，真正的核心是组件的单元测试。组件测试使用的方法是 testWidgets ，需要将组件放入到 MaterialApp 中，然后在 MaterialApp 中去 find 相应组件中的元素，接下来我们看一个比较简单的无状态组件的测试 。</p>
<h4 data-nodeid="694">无状态组件</h4>
<p data-nodeid="695">学习无状态组件的单元测试，我们选择上一课时中 article_detail 文件下的 article_content.dart 组件作为例子。在 test/article_detail 文件夹中创建 article_content_test.dart 文件，代码实现如下。</p>
<pre class="lang-dart" data-nodeid="696"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter_test/flutter_test.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/article_detail/article_content.dart'</span>;
<span class="hljs-keyword">void</span> main() {
  testWidgets(<span class="hljs-string">'test article content'</span>, (WidgetTester tester) <span class="hljs-keyword">async</span> {
    <span class="hljs-keyword">final</span> Widget testWidgets = ArticleContent(content: <span class="hljs-string">'test content'</span>);
    <span class="hljs-keyword">await</span> tester.pumpWidget(
        <span class="hljs-keyword">new</span> MaterialApp(
            home: testWidgets
        )
    );
    expect(find.text(<span class="hljs-string">'test content'</span>), findsOneWidget);
    expect(find.byWidget(testWidgets), findsOneWidget);
  });
}
</code></pre>
<ul data-nodeid="697">
<li data-nodeid="698">
<p data-nodeid="699">代码的前 2 行引入相应的组件库和测试库，第 4 行引入需要被测试的组件 article_content ；</p>
</li>
<li data-nodeid="700">
<p data-nodeid="701">在 main 函数中使用 testWidgets 来测试组件，testWidgets 也有两个参数，第一个是测试描述，第二个是一个执行函数，函数会自带一个组件测试对象 tester ；</p>
</li>
<li data-nodeid="702">
<p data-nodeid="703">在测试过程中需要将被测试的组件插入到 MaterialApp ，因此这里需要使用到tester.pumpWidget 方法，代码在第 9 行中体现；因为这是一个异步方法，因此需要函数使用 async ，并且这里需要使用 await 来等待执行完成；</p>
</li>
<li data-nodeid="704">
<p data-nodeid="705">使用 expect 来查询组件，findsOneWidget 来判断是否找到相应的组件。</p>
</li>
</ul>
<p data-nodeid="706">以上就是无状态组件的测试方法，由于上面的 article_content 内部只有一个 text 组件，因此单元测试比较简单。无状态组件可以验证组件是否存在，并且可以判断组件中的元素是否按照参数传入的值显示。</p>
<h4 data-nodeid="707">有状态组件</h4>
<p data-nodeid="708">有状态组件在组件测试部分与无状态组件一样，这里主要是介绍在组件触发更新后，如何保证界面显示正常与否。这里我们使用上一课时的 article_detail_like 作为测试例子。因为组件状态管理需要使用 Provider ，因此需要引入该模块。</p>
<pre class="lang-dart" data-nodeid="709"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter_test/flutter_test.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:provider/provider.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/model/like_num_model.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/article_detail/article_detail_like.dart'</span>;
</code></pre>
<p data-nodeid="710">接下来在 main 函数初始化状态模块 like_num_model，代码如下。</p>
<pre class="lang-dart" data-nodeid="711"><code data-language="dart"><span class="hljs-keyword">void</span> main() {
  <span class="hljs-keyword">final</span> LikeNumModel likeNumModel = LikeNumModel();
}
</code></pre>
<p data-nodeid="712">然后我们增加单纯的静态组件测试，这部分和无状态组件部分完全一致，代码如下。</p>
<pre class="lang-dart" data-nodeid="713"><code data-language="dart">testWidgets(<span class="hljs-string">'test article like widget'</span>, (WidgetTester tester) <span class="hljs-keyword">async</span> {
  <span class="hljs-keyword">final</span> Widget testWidgets = ArticleDetailLike();
  <span class="hljs-keyword">await</span> tester.pumpWidget(
      <span class="hljs-keyword">new</span> Provider&lt;<span class="hljs-built_in">int</span>&gt;.value(
          child: ChangeNotifierProvider.value(
            value: likeNumModel,
            child: MaterialApp(
                home: testWidgets
            ),
          )
      )
  );
  expect(find.byType(FlatButton), findsOneWidget);
  expect(find.byIcon(Icons.thumb_up), findsOneWidget);
  expect(find.text(<span class="hljs-string">'0'</span>), findsOneWidget);
});
</code></pre>
<p data-nodeid="714">与无状态组件测试唯一不同的是，我们需要使用 Provider 将 MaterialApp 封装起来。在代码中的第 13 行找 FlatButton 组件，第 14 行寻找 thumb_up icon ，第 15 行获取组件中的 Text 组件，并判断初始值为 0 。</p>
<p data-nodeid="715">接下来我们看下比较复杂的事件触发更新的测试部分逻辑。在这个例子的单元测试中，我们需要触发按钮点击操作，并且进行 rebuild 后，重新校验组件的正确性，代码如下。</p>
<pre class="lang-dart" data-nodeid="716"><code data-language="dart">testWidgets(<span class="hljs-string">'test article like widget when like action'</span>, (WidgetTester tester) <span class="hljs-keyword">async</span> {
  <span class="hljs-keyword">final</span> Widget testWidgets = ArticleDetailLike();
  <span class="hljs-keyword">await</span> tester.pumpWidget(
      <span class="hljs-keyword">new</span> Provider&lt;<span class="hljs-built_in">int</span>&gt;.value(
          child: ChangeNotifierProvider.value(
            value: likeNumModel,
            child: MaterialApp(
                home: testWidgets
            ),
          )
      )
  );
  <span class="hljs-keyword">await</span> tester.tap(find.byType(FlatButton));
  <span class="hljs-keyword">await</span> Future.microtask(tester.pump);
  expect(find.text(<span class="hljs-string">'1'</span>), findsOneWidget);
});
</code></pre>
<p data-nodeid="717">代码中的第 13 行就是找到 FlatButton 并且触发其点击操作，使用的是 tester.tap 方法，在触发后需要等待组件重新更新，因此需要使用 Future.microtask 来触发等待更新完成，完成后再校验组件中的点赞数是否更新，在上面的 17 行中使用 expect 再次判断。</p>
<h3 data-nodeid="718">综合实践</h3>
<p data-nodeid="719">以上就囊括了所有的单元测试的写法，由于断言 find 的方法还存在其他比较多的用法，这里就不复制过来，具体详细的内容，大家可以前往<a href="https://api.flutter.dev/flutter/flutter_test/CommonFinders-class.html" data-nodeid="821">官方文档</a>去查询。</p>
<p data-nodeid="720">接下来大家需要将上一课时的所有的组件使用本课时的知识点，覆盖到所有的单元测试，写完以后大家可以对比或者参考我们 github 上的源码。</p>
<p data-nodeid="721">这里也补充下，因为涉及图片组件，为了避免图片组件在测试加载过程中的异常问题，这里需要使用第三方库 image_test_utils ，下面是一个使用该组件的例子。</p>
<pre class="lang-dart" data-nodeid="722"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter_test/flutter_test.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:image_test_utils/image_test_utils.dart'</span>;

<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/util/struct/article_summary_struct.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/home_page/article_summary.dart'</span>;

<span class="hljs-keyword">void</span> main() {
&nbsp; <span class="hljs-comment">/// <span class="markdown">帖子概要描述信息</span></span>
&nbsp; <span class="hljs-keyword">final</span> ArticleSummaryStruct articleInfo = ArticleSummaryStruct(
&nbsp; &nbsp; &nbsp; <span class="hljs-string">'你好，交个朋友'</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">'我是一个小可爱，很长的一个测试看看效果，会换行吗'</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">'https://i.pinimg.com/originals/e0/64/4b/e0644bd2f13db50d0ef6a4df5a756fd9.png'</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-number">20</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-number">30</span>);

&nbsp; testWidgets(<span class="hljs-string">'test article summary'</span>, (WidgetTester tester) <span class="hljs-keyword">async</span> {
&nbsp; &nbsp; provideMockedNetworkImages(() <span class="hljs-keyword">async</span> {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> Widget testWidgets = ArticleSummary(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; title: articleInfo.title,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; summary: articleInfo.summary,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; articleImage: articleInfo.articleImage
&nbsp; &nbsp; &nbsp; );
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">await</span> tester.pumpWidget(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> MaterialApp(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; home: testWidgets
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; )
&nbsp; &nbsp; &nbsp; );

&nbsp; &nbsp; &nbsp; expect(find.text(<span class="hljs-string">'你好，交个朋友'</span>), findsOneWidget);
&nbsp; &nbsp; &nbsp; expect(find.text(<span class="hljs-string">'我是一个小可爱，很长的一个测试看看效果，会换行吗'</span>), findsOneWidget);

&nbsp; &nbsp; &nbsp; expect(find.byWidget(testWidgets), findsOneWidget);
&nbsp; &nbsp; });
&nbsp; });
}
</code></pre>
<p data-nodeid="723">主要看代码的第 22 行，需要将整个测试代码使用 provideMockedNetworkImages 函数来执行，这样就不会出现异常情况了。</p>
<h3 data-nodeid="724">总结</h3>
<p data-nodeid="725">以上就是本课时的所有内容，学完本课时你需要掌握 Struct、Model、无状态和有状态组件的单元测试写法。</p>
<p data-nodeid="726">下一课时我将把我们基础部分的所有基础知识汇总会一个脚手架，规范和统一基础模块。谢谢。</p>
<p data-nodeid="727" class="te-preview-highlight"><a href="https://github.com/love-flutter/flutter-column" data-nodeid="835">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### **河：
> 老师，请问一下单元测试的作用是什么，平时开发接上数据直接调不是更快吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果你形容当前快的话，那是肯定的。但是你放长远来看，如果你这个组件或者代码需要维护，比如新需求，涉及到这部分那就涉及到重构或者新增功能。如果这时候你没有单元测试，那是不是又需要把原来自测的逻辑走一遍，如果你有单元测试，那是不是可以直接跑一遍原来旧的测试用例，这样重构或者改造引来的问题都会很少。

##### **佳：
> 请问老师我运行flutter test之后会报错，内容如下：Failed to load "D:\flutterDaemon\overAll\test\util\struct\comment_info_struct_test.dart":Shell subprocess crashed with unexpected exit code -1073740791 before connecting to test harness.Test: D:\flutterDaemon\overAll\test\util\struct\comment_info_struct_test.dartShell: D:\flutter_windows_1.17.0-stable\flutter\bin\cache\artifacts\engine\windows-x64\flutter_tester.exe

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你运行下08课时里面的代码，记得运行的时候要在项目根目录。其次你换个最新的flutter版本，看官网在某个版本会出现运行 flutter test 在windows下会crash的情况，说已经修复。目前最新的是1.22，你试试。

##### **伟：
> 老师，这个Future.microtask的参数值跟tester.pump的返回值类型不一致怎么解决

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我没看到你具体的代码，如果不一致，检查下是否有多个地方运行这条测试用例，两边数据都进行了修改，导致了两个数据不一样。你可以在代码逻辑中增加调试信息，看下是哪部分对数据进行了修改导致两者不一致。

