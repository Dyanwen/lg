<p data-nodeid="35621" class="">上一课时我们已经创建好了项目的基础框架结构，其中有一个 router.dart 文件，该文件的作用是实现 App 内的一个路由管理和跳转。本课时是基于该功能模块，着重介绍如何实现 App 内外的路由跳转。</p>
<h3 data-nodeid="35622">Scheme</h3>
<p data-nodeid="35623">在介绍路由跳转实现之前，我们先来了解下 Scheme 的概念，Scheme 是一种 App 内跳转协议，通过 Scheme 协议在 APP 内实现一些页面的互相跳转。一般可以使用以下格式协议。</p>
<pre class="lang-java" data-nodeid="35624"><code data-language="java">[scheme]:<span class="hljs-comment">//[host]/[path]?[query]</span>
</code></pre>
<p data-nodeid="35625">这种格式不仅可以使用在 App 内部实现可跳转，还可以适用于外部拉起 App 指定页面的功能。内部跳转类似于前端的页面路由，只不过前端的页面路由是直接用链接来处理，在 App 中是无法像前端一样能够使用链接实现路由管理。外部跳转则需要分别从 Android 和 iOS 来介绍，其中主要的不同点是一些平台配置，下面我们先来看看内部如何实现路由跳转。</p>
<h3 data-nodeid="35626">内部跳转</h3>
<p data-nodeid="35627">按照上面的协议规则，我们将 Scheme 设置为项目 App 名字 tyfApp ，由于是 App 跳转 host 可以不设置，path 为具体的 pages 页面，query 则为 pages 需要的参数，举例如下。</p>
<pre class="lang-java" data-nodeid="35628"><code data-language="java">tyfapp:<span class="hljs-comment">//userPageIndex?userId=1001</span>
</code></pre>
<h4 data-nodeid="35629">基础版本</h4>
<p data-nodeid="35630">根据这个规则，实现 router.dart 逻辑。首先需要 import 相应的 pages 页面，例如这里我们需要两个页面的跳转和一个 Web 页面。</p>
<pre class="lang-dart" data-nodeid="35631"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/pages/common/web_view_page.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/pages/home_page/index.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/pages/user_page/index.dart'</span>;
</code></pre>
<p data-nodeid="35632">web_view_page.dart 使用第三方库，在遇到 http 或者 https 的协议时，使用该页面去打开，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="35633"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter_webview_plugin/flutter_webview_plugin.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">通用跳转逻辑</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CommonWebViewPage</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">url 地址</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> url;
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  CommonWebViewPage({Key key, <span class="hljs-keyword">this</span>.url}) : <span class="hljs-keyword">super</span>(key: key);
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> WebviewScaffold(
      url: url,
      appBar: AppBar(
        backgroundColor: Colors.blue,
      ),
    );
  }
}
</code></pre>
<p data-nodeid="35634">使用第三方库 flutter_webview_plugin 来打开具体的网页地址，该组件一个是 url 具体的网址，一个是 appBar 包含 title 和主题信息。如果非网页地址，并符合页面规范的协议时，我们需要解析出跳转的页面以及页面需要的参数。</p>
<pre class="lang-dart" data-nodeid="35635"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">解析跳转的url，并且分析其内部参数</span></span>
<span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; _parseUrl(<span class="hljs-built_in">String</span> url) {
  <span class="hljs-keyword">if</span>(url.startsWith(appScheme)) {
    url = url.substring(<span class="hljs-number">9</span>);
  }
  <span class="hljs-built_in">int</span> placeIndex = url.indexOf(<span class="hljs-string">'?'</span>);
  <span class="hljs-keyword">if</span>(url == <span class="hljs-string">''</span> || url == <span class="hljs-keyword">null</span>) {
    <span class="hljs-keyword">return</span> {<span class="hljs-string">'action'</span>: <span class="hljs-string">'/'</span>, <span class="hljs-string">'params'</span>: <span class="hljs-keyword">null</span>};
  }
  <span class="hljs-keyword">if</span> (placeIndex &lt; <span class="hljs-number">0</span>) {
    <span class="hljs-keyword">return</span> {<span class="hljs-string">'action'</span>: url, <span class="hljs-string">'params'</span>: <span class="hljs-keyword">null</span>};
  }
  <span class="hljs-built_in">String</span> action = url.substring(<span class="hljs-number">0</span>, placeIndex);
  <span class="hljs-built_in">String</span> paramStr = url.substring(placeIndex + <span class="hljs-number">1</span>);
  <span class="hljs-keyword">if</span> (paramStr == <span class="hljs-keyword">null</span>) {
    <span class="hljs-keyword">return</span> {<span class="hljs-string">'action'</span>: action, <span class="hljs-string">'params'</span>: <span class="hljs-keyword">null</span>};
  }
  <span class="hljs-built_in">Map</span> params = {};
  <span class="hljs-built_in">List</span>&lt;<span class="hljs-built_in">String</span>&gt; paramsStrArr = paramStr.split(<span class="hljs-string">'&amp;'</span>);
  <span class="hljs-keyword">for</span> (<span class="hljs-built_in">String</span> singleParamsStr <span class="hljs-keyword">in</span> paramsStrArr) {
    <span class="hljs-built_in">List</span>&lt;<span class="hljs-built_in">String</span>&gt; singleParamsArr = singleParamsStr.split(<span class="hljs-string">'='</span>);
    params[singleParamsArr[<span class="hljs-number">0</span>]] = singleParamsArr[<span class="hljs-number">1</span>];
  }
  <span class="hljs-keyword">return</span> {<span class="hljs-string">'action'</span>: action, <span class="hljs-string">'params'</span>: params};
}
</code></pre>
<p data-nodeid="35636">上面这段代码和前端解析 url 的代码逻辑完全一致，使用 ? 来分割 path 和参数两部分，再使用 &amp; 来获取参数的 key 和 value。解析出 path 和页面参数后，我们需要根据具体的 path 来跳转到相应的组件页面，并将解析出来的页面参数带入到组件中，代码如下。</p>
<pre class="lang-dart" data-nodeid="35637"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">根据url处理获得需要跳转的action页面以及需要携带的参数</span></span>
Widget _getPage(<span class="hljs-built_in">String</span> url, <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; urlParseRet) {
  <span class="hljs-keyword">if</span> (url.startsWith(<span class="hljs-string">'https://'</span>) || url.startsWith(<span class="hljs-string">'http://'</span>)) {
    <span class="hljs-keyword">return</span> CommonWebViewPage(url: url);
  } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(url.startsWith(appScheme)) {
    <span class="hljs-comment">// 判断是否解析出 path action，并且能否在路由配置中找到</span>
    <span class="hljs-built_in">String</span> pathAction = urlParseRet[<span class="hljs-string">'action'</span>].toString();
    <span class="hljs-keyword">switch</span> (pathAction) {
      <span class="hljs-keyword">case</span> <span class="hljs-string">"homepage"</span>: {
        <span class="hljs-keyword">return</span> _buildPage(HomePageIndex());
      }
      <span class="hljs-keyword">case</span> <span class="hljs-string">"userpage"</span>: {
        <span class="hljs-comment">// 必要性检查，如果没有参数则不做任何处理</span>
        <span class="hljs-keyword">if</span>(urlParseRet[<span class="hljs-string">'params'</span>][<span class="hljs-string">'user_id'</span>].toString() == <span class="hljs-keyword">null</span>) {
          <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
        }
        <span class="hljs-keyword">return</span> _buildPage(
            UserPageIndex(
                userId: urlParseRet[<span class="hljs-string">'params'</span>][<span class="hljs-string">'user_id'</span>].toString()
            )
        );
      }
      <span class="hljs-keyword">default</span>: {
        <span class="hljs-keyword">return</span> _buildPage(HomePageIndex());
      }
    }
  }
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
}
</code></pre>
<p data-nodeid="35638">首先就是判断是否以 http 和 https 开头，如果是则使用网页打开，如果不是则继续判断是否符合 App Scheme 信息，符合则解析出相应的 path 和参数信息，并且根据 path 调用对应的页面组件，打开相应的页面信息。如果匹配不到或者不符合 App Scheme 则不做任何处理。</p>
<p data-nodeid="35639">最后为该 Router 类添加一个可外部调用的函数，执行页面跳转，代码如下。</p>
<pre class="lang-dart" data-nodeid="35640"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">执行页面跳转</span></span>
<span class="hljs-keyword">void</span> push(BuildContext context, <span class="hljs-built_in">String</span> url) {
  <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; urlParseRet = _parseUrl(url);
  <span class="hljs-comment">// 不同页面，则跳转</span>
  Navigator.push(context, MaterialPageRoute(builder: (context) {
    <span class="hljs-keyword">return</span> _getPage(url, urlParseRet);
  }));
}
</code></pre>
<p data-nodeid="35641">上面逻辑会存在一些问题，主要问题是没有考虑到当前页面，无论什么情况下都会打开一个新的页面，这样会很耗费机器资源，接下来我就介绍下如何优化这块逻辑。</p>
<h4 data-nodeid="35642">进阶版本</h4>
<p data-nodeid="35643">进阶版本的目的是判断当前是否有打开页面，如果打开了页面则替换和刷新旧页面，如果没有则打开新的页面。分为以下两点来分析：</p>
<ol data-nodeid="35644">
<li data-nodeid="35645">
<p data-nodeid="35646">了解页面标识，具体打开的页面路由名称；</p>
</li>
<li data-nodeid="35647">
<p data-nodeid="35648">判断当前打开的页面，如果已经打开则更新，未打开则新建窗口；</p>
</li>
</ol>
<p data-nodeid="35649">首先，我们需要使用到 Flutter 路由注册功能，我们需要修改 main.dart 中的代码，在 build MaterialApp 组件中增加两个函数，代码如下：</p>
<pre class="lang-dart" data-nodeid="35650"><code data-language="dart"><span class="hljs-keyword">return</span> MaterialApp(
    title: <span class="hljs-string">'Two You'</span>, <span class="hljs-comment">// APP 名字</span>
    debugShowCheckedModeBanner: <span class="hljs-keyword">false</span>,
    theme: ThemeData(
      primarySwatch: Colors.blue, <span class="hljs-comment">// APP 主题</span>
    ),
    routes: Router().registerRouter(),
    home: Scaffold(
        appBar: AppBar(
          title: Text(<span class="hljs-string">'Two You'</span>), <span class="hljs-comment">// 页面名字</span>
        ),
        body: Center(
          child: Entrance(),
        )));
</code></pre>
<p data-nodeid="35651">上面代码中的第 7 行是注册路由名字，我们看下 Router 中的实现。</p>
<pre class="lang-dart" data-nodeid="35652"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">注册路由事件</span></span>
<span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, Widget <span class="hljs-built_in">Function</span>(BuildContext)&gt; registerRouter () {
  <span class="hljs-keyword">return</span> {
    <span class="hljs-string">'homepage'</span> : (context) =&gt; _buildPage(HomePageIndex()),
    <span class="hljs-string">'userpage'</span> : (context) =&gt; _buildPage(UserPageIndex())
  };
}
</code></pre>
<p data-nodeid="35653">按照这个方式注册其他的页面信息即可，接下来我们着重看下 push 的方法。</p>
<pre class="lang-dart" data-nodeid="35654"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">执行页面跳转</span></span>
<span class="hljs-keyword">void</span> push(BuildContext context, <span class="hljs-built_in">String</span> url) {
  <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; urlParseRet = _parseUrl(url);
  Navigator.pushNamedAndRemoveUntil(
      context,
      urlParseRet[<span class="hljs-string">'action'</span>].toString(), (route) {
            <span class="hljs-keyword">if</span>(route.settings.name ==
                urlParseRet[<span class="hljs-string">'action'</span>].toString()) {
              <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
            }
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
          }, arguments: urlParseRet[<span class="hljs-string">'params'</span>]);
  }
}
</code></pre>
<p data-nodeid="35655">在原来的基础上我们修改了方法，使用到了 pushNamedAndRemoveUntil 方法，并且在第二个回调参数中判断是否为当前页面，如果是则返回 false，否则返回 true。这种方式有个缺点就是在具体的 pages 页面中不能直接通过构造函数去获取参数，必须使用下面的方式。</p>
<pre class="lang-dart" data-nodeid="35656"><code data-language="dart"><span class="hljs-meta">@override</span>
Widget build(BuildContext context) {
  <span class="hljs-built_in">Map</span> dataInfo = JsonConfig.objectToMap(
      ModalRoute.of(context).settings.arguments
  );
  <span class="hljs-comment">// <span class="hljs-doctag">TODO:</span> implement build</span>
  <span class="hljs-keyword">return</span> Text(<span class="hljs-string">'I am use page <span class="hljs-subst">${dataInfo[<span class="hljs-string">'userId'</span>]}</span>'</span>);
}
</code></pre>
<p data-nodeid="35657">获取的方式是第 3 到 5 行，这段逻辑还必须在 build 中，存在一定缺陷，现阶段还没有找到其他的解决方案，后续有解决方案再通过源码进行更新。</p>
<p data-nodeid="35658">以上就是整个 router.dart 的实现逻辑，这样就可以在 APP 内的页面实现跳转，接下来我们看看如何在 App 外也能使用这个 Scheme，拉起 App。</p>
<h3 data-nodeid="35659">外部跳转</h3>
<p data-nodeid="35660">该功能的实现，需要使用 <a href="https://pub.dev/packages/uni_links" data-nodeid="35754">uni_links</a> 第三方库来协助完成外部页面的 Scheme，在 pubspec.yaml 中增加依赖，然后更新本地库文件。由于 Android 和 iOS 在配置上会有点区别，因此这里分别来介绍。</p>
<h4 data-nodeid="35661">Android 流程</h4>
<p data-nodeid="35662">在项目中找到这个路径下的文件</p>
<pre class="lang-java" data-nodeid="35663"><code data-language="java">android/app/src/main/AndroidManifest.xml
</code></pre>
<p data-nodeid="35664">在配置的 application 下的 activity 内增加如下配置：</p>
<pre class="lang-xml" data-nodeid="35665"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">intent-filter</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">action</span> <span class="hljs-attr">android:name</span>=<span class="hljs-string">"android.intent.action.VIEW"</span>/&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">category</span> <span class="hljs-attr">android:name</span>=<span class="hljs-string">"android.intent.category.DEFAULT"</span>/&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">category</span> <span class="hljs-attr">android:name</span>=<span class="hljs-string">"android.intent.category.BROWSABLE"</span>/&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">data</span>
        <span class="hljs-attr">android:scheme</span>=<span class="hljs-string">"tyfapp"</span>/&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">intent-filter</span>&gt;</span>
</code></pre>
<p data-nodeid="35666">第 6 行代码就是声明这个 App 的 Scheme 的协议。</p>
<h4 data-nodeid="35667">iOS 流程</h4>
<p data-nodeid="35668">在项目中找到这个路径下的文件</p>
<pre class="lang-java" data-nodeid="35669"><code data-language="java">ios/Runner/info.plist
</code></pre>
<p data-nodeid="35670">在 dict 内增加下面的配置：</p>
<pre class="lang-xml" data-nodeid="35671"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">key</span>&gt;</span>CFBundleURLTypes<span class="hljs-tag">&lt;/<span class="hljs-name">key</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">array</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">dict</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">key</span>&gt;</span>CFBundleTypeRole<span class="hljs-tag">&lt;/<span class="hljs-name">key</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">string</span>&gt;</span>Editor<span class="hljs-tag">&lt;/<span class="hljs-name">string</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">key</span>&gt;</span>CFBundleURLName<span class="hljs-tag">&lt;/<span class="hljs-name">key</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">string</span>&gt;</span>Two You<span class="hljs-tag">&lt;/<span class="hljs-name">string</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">key</span>&gt;</span>CFBundleURLSchemes<span class="hljs-tag">&lt;/<span class="hljs-name">key</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">array</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">string</span>&gt;</span>tyfapp<span class="hljs-tag">&lt;/<span class="hljs-name">string</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">array</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">dict</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">array</span>&gt;</span>
</code></pre>
<p data-nodeid="35672">其中的第 9 行声明 App 的 Scheme。<br>
以上就完成了基础的配置，接下来我们就使用 uni_links 来实现 Scheme 的监听。</p>
<h4 data-nodeid="35673">Uni_links 实现外部跳转</h4>
<p data-nodeid="35674">首先我们在 pages 目录下新建一个主入口文件 entrance.dart ，该文件需要设计为一个有状态类组件。在组件中最关键的是监听获取到打开 App 的链接地址，实现的方式如下代码。</p>
<pre class="lang-dart" data-nodeid="35675"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">使用[String]链接实现</span></span>
Future&lt;<span class="hljs-keyword">void</span>&gt; initPlatformStateForStringUniLinks() <span class="hljs-keyword">async</span> {
  <span class="hljs-built_in">String</span> initialLink;
  <span class="hljs-comment">// Platform messages may fail, so we use a try/catch PlatformException.</span>
  <span class="hljs-keyword">try</span> {
    initialLink = <span class="hljs-keyword">await</span> getInitialLink();
    <span class="hljs-keyword">if</span> (initialLink != <span class="hljs-keyword">null</span>) {
      <span class="hljs-comment">//  跳转到指定页面</span>
      router.push(context, initialLink);
    }
  } <span class="hljs-keyword">on</span> PlatformException {
    initialLink = <span class="hljs-string">'Failed to get initial link.'</span>;
  } <span class="hljs-keyword">on</span> FormatException {
    initialLink = <span class="hljs-string">'Failed to parse the initial link as Uri.'</span>;
  }
  <span class="hljs-comment">// Attach a listener to the links stream</span>
  _sub = getLinksStream().listen((<span class="hljs-built_in">String</span> link) {
    <span class="hljs-keyword">if</span> (!mounted || link == <span class="hljs-keyword">null</span>) <span class="hljs-keyword">return</span>;
    <span class="hljs-comment">//  跳转到指定页面</span>
    router.push(context, link);
  }, onError: (<span class="hljs-built_in">Object</span> err) {
    <span class="hljs-keyword">if</span> (!mounted) <span class="hljs-keyword">return</span>;
  });
</code></pre>
<p data-nodeid="35676">其中第 6 行是处理在外部直接拉起 App 的业务逻辑，第 17 行则表示当前 App 处于打开状态，监听外部拉起事件，监听变化后处理相应的跳转逻辑。由于组件中有一个监听事件，为了避免组件被销毁后还在监听，因此需要在组件销毁阶段移除监听事件，代码如下：</p>
<pre class="lang-dart" data-nodeid="35677"><code data-language="dart"><span class="hljs-meta">@override</span>
<span class="hljs-keyword">void</span> dispose() {
  <span class="hljs-keyword">super</span>.dispose();
  <span class="hljs-keyword">if</span> (_sub != <span class="hljs-keyword">null</span>) _sub.cancel();
}
</code></pre>
<p data-nodeid="35678">为了验证效果，使用了一个 <a href="https://love-flutter.github.io/test-page/index.html" data-nodeid="35776">github 上创建的测试页面</a>。接下来我们运行下程序，然后在手机模拟器中打开测试页面，可以看到如图 1 所示的效果。</p>
<p data-nodeid="35679"><img src="https://s0.lgstatic.com/i/image/M00/2F/E1/Ciqc1F8IDbaAIpJ7AESu3z7EomU793.gif" alt="1.gif" data-nodeid="35780"></p>
<div data-nodeid="35680"><p style="text-align:center">图 1 Scheme 实现运行效果</p></div>
<p data-nodeid="35681">以上就实现了 Scheme 可以直接在内外部使用的跳转逻辑。不过 Scheme 在 App 外部存在一些体验方面的问题，比如：</p>
<ul data-nodeid="35682">
<li data-nodeid="35683">
<p data-nodeid="35684">当需要被拉起的 App 没有被安装时，这个链接就不会生效；</p>
</li>
<li data-nodeid="35685">
<p data-nodeid="35686">在大部分 App 内 Scheme 是被禁用的，因此在用户体验的时候会非常差；</p>
</li>
<li data-nodeid="35687">
<p data-nodeid="35688">注册的 Scheme 相同导致冲突；</p>
</li>
</ul>
<p data-nodeid="35689">为了解决上述问题，Andorid 和 iOS 都提供了一套解决方案，在 Android 叫作 App link / Deep links ，在 iOS 叫作 Universal Links / Custom URL schemes。解决的方案就是在未安装 App 时可提供网页跳转，其次可以使用 https 和 http 域名链接的方式来进一步提升唯一性。</p>
<p data-nodeid="35690"><strong data-nodeid="35789">App link / Deep links</strong></p>
<p data-nodeid="35691">应用链接仅适用于 https 方案，并且需要指定的主机以及托管文件 assetlinks.json，该配置文件可参考如下：</p>
<pre class="lang-json" data-nodeid="35692"><code data-language="json">[{
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">"relation"</span>: [<span class="hljs-string">"delegate_permission/common.handle_all_urls"</span>],
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">"target"</span>: {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"namespace"</span>: <span class="hljs-string">"android_app"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"package_name"</span>: <span class="hljs-string">"com.example"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"sha256_cert_fingerprints"</span>:
&nbsp; &nbsp; &nbsp; &nbsp; [<span class="hljs-string">"14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"</span>]
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }]
</code></pre>
<ul data-nodeid="35693">
<li data-nodeid="35694">
<p data-nodeid="35695">package_name，在应用的 build.gradle 文件中声明的应用 ID；</p>
</li>
<li data-nodeid="35696">
<p data-nodeid="35697">sha256_cert_fingerprints：应用签名证书的 SHA256 指纹，你可以利用 Java 密钥工具。</p>
</li>
</ul>
<p data-nodeid="35698">配置好该文件后，同样是修改下面路径下的文件。</p>
<pre class="lang-java" data-nodeid="35699"><code data-language="java">android/app/src/main/AndroidManifest.xml
</code></pre>
<p data-nodeid="35700">在配置的 application 下的 activity 内增加如下配置：</p>
<pre class="lang-xml" data-nodeid="35701"><code data-language="xml"><span class="hljs-comment">&lt;!-- App Links --&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">intent-filter</span> <span class="hljs-attr">android:autoVerify</span>=<span class="hljs-string">"true"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">action</span> <span class="hljs-attr">android:name</span>=<span class="hljs-string">"android.intent.action.VIEW"</span> /&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">category</span> <span class="hljs-attr">android:name</span>=<span class="hljs-string">"android.intent.category.DEFAULT"</span> /&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">category</span> <span class="hljs-attr">android:name</span>=<span class="hljs-string">"android.intent.category.BROWSABLE"</span> /&gt;</span>
        <span class="hljs-comment">&lt;!-- Accepts URIs that begin with https://YOUR_HOST --&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">data</span>
          <span class="hljs-attr">android:scheme</span>=<span class="hljs-string">"https"</span>
          <span class="hljs-attr">android:host</span>=<span class="hljs-string">"[YOUR_HOST]"</span> /&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">intent-filter</span>&gt;</span>
</code></pre>
<p data-nodeid="35702">具体的过程，你可以在线上项目开发过程中尝试应用。</p>
<p data-nodeid="35703"><strong data-nodeid="35805">Universal Links / Custom URL schemes</strong></p>
<p data-nodeid="35704">该方法也是需要一个主机域名来启动应用，因此需要服务的一个在线配置，例如：<a href="https://www.example.test" data-nodeid="35809">https://www.example.test</a>/apple-app-site-association 获取 apple-app-site-association 的配置文件如下。</p>
<pre class="lang-java" data-nodeid="35705"><code data-language="java">{
    <span class="hljs-string">"applinks"</span>: {
        <span class="hljs-string">"apps"</span>: [],
        <span class="hljs-string">"details"</span>: [
            {
                <span class="hljs-string">"appID"</span>: <span class="hljs-string">"8LX3M43WHV.me.gexiao.me"</span>,
                <span class="hljs-string">"paths"</span>: [ <span class="hljs-string">"/*"</span> ]
            }
        ]
    }
}
</code></pre>
<p data-nodeid="35706">同样我们需要修改下面路径的文件。</p>
<pre class="lang-java" data-nodeid="35707"><code data-language="java">ios/Runner/info.plist
</code></pre>
<p data-nodeid="35708">在 dict 内增加下面的配置：</p>
<pre class="lang-xml" data-nodeid="35709"><code data-language="xml"> <span class="hljs-tag">&lt;<span class="hljs-name">key</span>&gt;</span>com.apple.developer.associated-domains<span class="hljs-tag">&lt;/<span class="hljs-name">key</span>&gt;</span>
 <span class="hljs-tag">&lt;<span class="hljs-name">array</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">string</span>&gt;</span>applinks:[YOUR_HOST]<span class="hljs-tag">&lt;/<span class="hljs-name">string</span>&gt;</span>
 <span class="hljs-tag">&lt;/<span class="hljs-name">array</span>&gt;</span>
</code></pre>
<p data-nodeid="35710">以上就是外部跳转的实现方案，实现外部跳转的 App Links 和 Universal Link 功能，由于需要域名部署，我这里就没有实际应用，具体你可以在项目开发中尝试。</p>
<h3 data-nodeid="35711">总结</h3>
<p data-nodeid="35712">本课时介绍了在 Flutter 中路由跳转以及外部 Scheme 启动 App 的方法，最后简单介绍了 App Links 和 Universal Link 的知识点。学完本课时你需要掌握开发 Flutter 路由跳转基础技巧，并且能够应用 uni_links 库实现外部 Scheme 启动 App 功能。</p>
<p data-nodeid="38825">下一课时我将介绍 Flutter 中各种导航栏的设计，我会在本课时的基础上增加导航栏功能，其次我也会实现首页和个人页面的代码逻辑。</p>
<p data-nodeid="51691" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="51694">点击链接，查看本课时源码。</a></p>

---

### 精选评论

##### *聪：
> Uri.parse();去解析一个URI，而不.建议用字符串截取分割等操作

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢提醒，非常好的建议。

##### **君：
> Android app links 在国内能识别么 没梯子也可以？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以的，不用梯子，需要有https域名以及用户端的android m之后的版本就行了。

##### *蛋：
> 其实 iOS 不需要添加这一步：

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢提醒，想了解是哪一部分，是指外部跳转的iOS部分的配置修改吗，如果真的不需要，我自行验证确认下。避免给读者造成误解。

##### *萍：
> onTap: (){goToWebView;},和 onTap: goToWebView,有什么区别？为啥goToWebView方法有参数的时候初始化就立即调用，不用等到执行onTap点击事件

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 两者没有太大的区别，前者只是用闭包进行了封装，如果被调用是一个无参数的函数可以直接写函数名，如果非函数或者是一个带参数的函数，比如我们要在点击后print一个数据，那么就必须要用(){}来封装。onTap: goToWebView()调用是会提示失败的哦，因为非一个 void Function返回结构，onTap需要一个void Function。如果有源代码可以提供下，我也了解下，因为我自己试了下是不行的。

##### **泽：
> 路由rote 打卡学习

