<p data-nodeid="605" class="">本课时将实现一个通用的 Flutter 异常上报模块，为线上运营提供上报与监控安全保障，其次借助 Sentry 平台实现异常告警和管理功能。我们先来看下 App 的线上安全包括哪些点。</p>
<h3 data-nodeid="606">线上安全运营</h3>
<p data-nodeid="6114" class="">在上线每个客户端版本时，我们都要有一个 crash 标准，由于客户端发布后是无法回滚的，因此在发布时需要使用数据来辅助判断，这里就需要使用 crash 率来辅助判断是否需要进一步灰度更多用户。要做到这点，就需要将客户端异常的报错问题，上报到平台。Android 和 iOS 都有类似的功能模块，在 <a href="https://flutter.cn/docs/cookbook/maintenance/error-reporting" data-nodeid="6118">Flutter 官网</a>也介绍了相应的理论方法，大家可以前往官网进行了解。写这篇专栏的目的是希望将该功能作为通用模块，并且实践该过程。</p>




<p data-nodeid="608">接下来我们就看看，如何利用官网的理论知识，将该功能封装为一个通用的异常上报模块。</p>
<h3 data-nodeid="609">通用上报模块</h3>
<p data-nodeid="610">根据官网的文档，我们将该过程分为三个步骤：</p>
<ol data-nodeid="611">
<li data-nodeid="612">
<p data-nodeid="613">修改 main.dart 中的 main 函数；</p>
</li>
<li data-nodeid="614">
<p data-nodeid="615">异常捕获；</p>
</li>
<li data-nodeid="616">
<p data-nodeid="617">捕获内容上报。</p>
</li>
</ol>
<p data-nodeid="618">接下来我们就分别看看三个过程中的实现原理。</p>
<h4 data-nodeid="619">第一步：修改 main.dart 中 main 函数</h4>
<p data-nodeid="620">在我们之前项目代码逻辑的 main 函数中，都是使用下面的代码方式来启动运行 Flutter 项目。</p>
<pre class="lang-dart" data-nodeid="4911"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">App 核心入口文件 </span></span>
<span class="hljs-keyword">void</span> main() { 
  runApp(MyApp()); 
}
</code></pre>




<p data-nodeid="622">由于需要 catch 异常，因此我们将 runApp 执行放到异常捕获模块中去执行，修改如下的方式。</p>
<pre class="lang-dart" data-nodeid="3723"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/tools/app_sentry.dart'</span>; 
<span class="hljs-comment">/// <span class="markdown">App 核心入口文件 </span></span>
<span class="hljs-keyword">void</span> main() { 
  AppSentry.runWithCatchError(MyApp()); 
}
</code></pre>




<p data-nodeid="624">这里引入了通用异常捕获模块，后续你在项目中，也可以直接使用该模块实现上报，接下来我们来看下 AppSentry 的实现异常捕获逻辑。</p>
<h4 data-nodeid="625">第二步：异常捕获</h4>
<p data-nodeid="626">该功能的实现会应用到 FlutterError 和 runZonedGuarded 两个知识点。</p>
<p data-nodeid="627">在 Flutter 中可以通过 FlutterError 来捕获到运行期间的错误，包括构建期间、布局期间和绘制期间。</p>
<p data-nodeid="628">runZonedGuarded 则是使用 Zone.fork 创建一片新的区域去运行代码逻辑，也就是 runApp，当遇到错误时会执行其回调函数 onError，其次如果在项目使用了 Zone.current.handleUncaughtError 也会将错误抛出执行 onError 逻辑，具体我们看下代码，如下：</p>
<pre class="lang-dart" data-nodeid="629"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">catch 组件异常 </span></span>
<span class="hljs-comment">/// 
<span class="markdown">/// 开发模式下，本地打印，上线时则调用 sentry 平台 </span></span>
<span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> runWithCatchError(Widget appMain) { 
  <span class="hljs-comment">// 捕获并上报 Flutter 异常 </span>
  FlutterError.onError = (FlutterErrorDetails details) <span class="hljs-keyword">async</span> { 
    <span class="hljs-keyword">if</span> (!inProduction) { 
      FlutterError.dumpErrorToConsole(details); 
    } <span class="hljs-keyword">else</span> { 
      Zone.current.handleUncaughtError(details.exception, details.stack); 
    } 
  }; 
  runZonedGuarded&lt;Future&lt;<span class="hljs-built_in">Null</span>&gt;&gt;(() <span class="hljs-keyword">async</span> { 
    runApp(appMain); 
  }, (error, stackTrace) <span class="hljs-keyword">async</span> { 
    <span class="hljs-keyword">await</span> _reportError(error, stackTrace); 
  }); 
}
</code></pre>
<p data-nodeid="630">FlutterError.onError 来捕获异常，这里会判断是否在正式环境，如果是则在本地打印错误日志，如果不是则去执行 runZonedGuarded onError 逻辑。在 runZonedGuarded 代码中执行 runApp，遇到异常时则调用 _reportError 实现错误上报，接下来我们就看下 _reportError 的实现部分。</p>
<h4 data-nodeid="631">第三步：异常上报</h4>
<p data-nodeid="632">上面已经获取到了客户端的具体异常信息，接下来我们利用 Sentry 平台将异常信息进行上报。应用 Sentry 平台前，需要去申请接入，具体接入方法在本课时的“Sentry 平台”中介绍，成功接入 Sentry 平台后，会提供一个 dsn。这里我们需要将其配置在该文件中，如下代码。因为 Sentry 是一个第三方库，因此需要在 pubspec.yaml 增加库依赖，并更新本地库。</p>
<pre class="lang-dart" data-nodeid="633"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">创建 SentryClient 用于将异常日志上报给 sentry 平台 </span></span>
<span class="hljs-keyword">final</span> SentryClient _sentry = SentryClient( 
  dsn: <span class="hljs-string">'https://f886adfd35e64062b01feb5e9a8723f6@o425523.ingest.sentry.io/5362342'</span>, 
); 
<span class="hljs-comment">/// <span class="markdown">判断当前环境类型 </span></span>
<span class="hljs-keyword">const</span> <span class="hljs-built_in">bool</span> inProduction = <span class="hljs-built_in">bool</span>.fromEnvironment(<span class="hljs-string">"dart.vm.product"</span>);
</code></pre>
<p data-nodeid="634">_sentry 就是利用 Sentry 第三方库创建的上报句柄，dsn 则是我们测试的上报标识，请大家测试时自己前往申请，不然无法看到自己的异常上报内容。上述代码中 inProduction 是用来获取是否为正式环境的逻辑。接下来我们看下上报逻辑，代码如下：</p>
<pre class="lang-dart" data-nodeid="635"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">上报异常的函数 </span></span>
<span class="hljs-keyword">static</span> Future&lt;<span class="hljs-keyword">void</span>&gt; _reportError(<span class="hljs-built_in">dynamic</span> error, <span class="hljs-built_in">dynamic</span> stackTrace) <span class="hljs-keyword">async</span> { 
  <span class="hljs-keyword">if</span> (!inProduction) { <span class="hljs-comment">// 判断是否为正式环境 </span>
    <span class="hljs-built_in">print</span>(stackTrace); 
  } 
  <span class="hljs-comment">// sentry 上报 </span>
  <span class="hljs-keyword">final</span> SentryResponse response = <span class="hljs-keyword">await</span> _sentry.captureException( 
    exception: error, 
    stackTrace: stackTrace, 
  ); 
  <span class="hljs-keyword">if</span> (response.isSuccessful) { 
    <span class="hljs-built_in">print</span>(<span class="hljs-string">'Success! Event ID: <span class="hljs-subst">${response.eventId}</span>'</span>); 
  } <span class="hljs-keyword">else</span> { 
    <span class="hljs-built_in">print</span>(<span class="hljs-string">'Failed to report to Sentry.io: <span class="hljs-subst">${response.error}</span>'</span>); 
  } 
}
</code></pre>
<p data-nodeid="636">首先还是判断是否为正式环境，如果不是则直接打印，也可以应用下面的代码打印本地日志。如果是正式环境，则使用 _sentry.captureException 来上报异常，最后打印是否执行成功，后面一段是可以去掉的，主要是为了在真机调试时候查看。</p>
<pre class="lang-dart" data-nodeid="637"><code data-language="dart">FlutterError.dumpErrorToConsole(FlutterErrorDetails(exception: error));
</code></pre>
<p data-nodeid="638">以上就完成了通用上报工具的代码，接下来我们来教大家如何注册 Sentry 并创建项目获得 dsn。</p>
<h3 data-nodeid="639">Sentry 平台</h3>
<p data-nodeid="640">我们来看下具体的创建步骤：</p>
<p data-nodeid="641">1.创建注册账户， <a href="https://sentry.io/signup/" data-nodeid="706">前往官网</a>，具体请按照你自己的信息填写，注册完成后，直接登录；<br>
2.创建项目，这个过程如果没有分组会让填写分组，按要求填写即可；<br>
3.上报测试，里面会有各种 SDK 的例子，唯独是没有 Flutter 的，你可以不关注，直接进入项目中或者进入个人页面，可以看到如图 1 的界面。</p>
<p data-nodeid="642"><img src="https://s0.lgstatic.com/i/image/M00/40/BC/Ciqc1F8zjOiAZrLtAAUYl4LuWZQ819.png" alt="image.png" data-nodeid="714"></p>
<div data-nodeid="643"><p style="text-align:center">图 1 Sentry 平台</p></div>
<p data-nodeid="644">4.按照图 2 的方法可以看到我们需要的 dsn，如果是新注册的，可以点击图 2 中的 Setup Sentry 快速找到；</p>
<p data-nodeid="645"><img src="https://s0.lgstatic.com/i/image/M00/40/BC/Ciqc1F8zjPmADPTqAAZbn--G0NI800.png" alt="image (1).png" data-nodeid="718"></p>
<div data-nodeid="646"><p style="text-align:center">图 2 dsn 查询指引</p></div>
<p data-nodeid="647">5.替换项目代码中的 dsn。</p>
<p data-nodeid="648">经过以上 5 个步骤就可以完成项目的接入，接下来我们就验证下上报是否正常。</p>
<h3 data-nodeid="649">上报测试</h3>
<p data-nodeid="650">为了验证该功能，我们需要特意去修改正常的代码改为异常。这里我们将左侧菜单“图片流”中的代码做如下修改：</p>
<pre class="lang-dart" data-nodeid="651"><code data-language="dart">Widget build(BuildContext context) { 
  <span class="hljs-keyword">if</span> (contentList == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 判断是否为空 </span>
    <span class="hljs-keyword">return</span> Loading(); 
  } 
  <span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; tmpList = []; 
  <span class="hljs-keyword">return</span> ListView.separated( 
    scrollDirection: Axis.vertical, 
    shrinkWrap: <span class="hljs-keyword">true</span>, 
    itemCount: contentList.length + <span class="hljs-number">1</span>, <span class="hljs-comment">// 增加异常 </span>
    ......
</code></pre>
<p data-nodeid="652">上面只是 pages/home_page/img_flow.dart 部分代码，其中代码第 10 行就是增加了异常，由于数组长度超出了限制，这样会导致在 ListView.separated 抛出数组边界异常。请注意该异常捕获逻辑只会在 release 模式下生效，因此需要正式 build app，利用上一课时的知识，我们将 App build 为一个 apk 文件，然后安装到 Android 手机。</p>
<p data-nodeid="653">成功安装后，打开 App 选择左侧菜单的“图片流”，这时候你会看到界面异常，大概过 1-2 分钟后，我们前往 Sentry 平台，选择我们创建项目，就会看到如下图 3 的上报内容。</p>
<p data-nodeid="654"><img src="https://s0.lgstatic.com/i/image/M00/40/C8/CgqCHl8zjR6AAojgAAKndALdwFQ016.png" alt="image (2).png" data-nodeid="731"></p>
<div data-nodeid="655"><p style="text-align:center">图 3 Sentry 异常例子</p></div>
<p data-nodeid="656">点击具体的上报内容，就可以看到详细的报错信息，这里面已经将报错的文件和具体函数都说明了，并打印了调用堆栈。你可以在提示信息中查看到具体的报错文件 img_flow.dart 以及具体的报错内容 RangeError ，通过文件以及报错详情，就可以非常容易的定位到具体问题了。</p>
<p data-nodeid="657">以上就是异常上报的工具模块，通过这种方式，我们就可以快速地收集问题并解决问题。由于本课时将 main.dart 进行了优化，因此这里也顺带介绍下重构的部分。</p>
<h3 data-nodeid="658">AppProvider</h3>
<p data-nodeid="659">为了减轻 main.dart 中的代码，使其更精简，我们将 Provider 部分的逻辑转移到一个新的工具模块中，这个工具在 util/tools/app_provider 中。而 main.dart 中从原来调用函数内部的 _getProviders 方法，修改为调用 AppProvider 中的 getProviders 方法，main.dart 代码如下：</p>
<pre class="lang-dart" data-nodeid="2535"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">App 核心入口文件 </span></span>
<span class="hljs-keyword">void</span> main() { 
  AppSentry.runWithCatchError(MyApp()); 
} 
<span class="hljs-comment">/// <span class="markdown">MyApp 核心入口界面 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyApp</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{ 
  <span class="hljs-comment">// This widget is the root of your application. </span>
  <span class="hljs-meta">@override</span> 
  Widget build(BuildContext context) { 
    <span class="hljs-keyword">return</span> FutureBuilder&lt;Widget&gt;( 
        future: AppProvider.getProviders( 
          context, 
          MaterialApp( 
              title: <span class="hljs-string">'Two You'</span>, <span class="hljs-comment">// APP 名字 </span>
              debugShowCheckedModeBanner: <span class="hljs-keyword">false</span>, 
              theme: ThemeData( 
                primarySwatch: Colors.blue, <span class="hljs-comment">// App 主题 </span>
              ), 
              routes: Router().registerRouter(), 
              home: Entrance()), 
        ), 
        builder: (BuildContext context, AsyncSnapshot&lt;Widget&gt; snapshot) { 
          <span class="hljs-keyword">if</span> (snapshot.error != <span class="hljs-keyword">null</span>) { 
            <span class="hljs-keyword">return</span> Container( 
              child: CommonError(), 
            ); 
          } 
          <span class="hljs-keyword">return</span> Container( 
            child: snapshot.data, 
          ); 
        }); 
  } 
}
</code></pre>







<p data-nodeid="661">代码就显得非常精简了，在代码中的第 12 行修改了调用方法，其次在第 23 行中，增加了组件异常处理。AppProvider 中的 getProviders 方法就是原来首页的逻辑，基本没有变化，大家可以自己前往 <a href="https://github.com/love-flutter/flutter-column" data-nodeid="745">github 源码</a>中查看。</p>
<h3 data-nodeid="662">总结</h3>
<p data-nodeid="663">本课时着重介绍了通用上报模块的实现方法，同时实践介绍了 Sentry 平台的应用，最后再简单介绍了 main.dart 中的重构逻辑。学完本课时后，需要掌握 Flutter 异常捕获处理方法，并会应用 Sentry 平台来协助分析异常问题。</p>
<p data-nodeid="664">下一课时我们将进入下一阶段，我们还将继续在 Two You App 上完善功能。</p>
<p data-nodeid="665" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="752">点击此链接查看本课时源码</a></p>

---

### 精选评论


