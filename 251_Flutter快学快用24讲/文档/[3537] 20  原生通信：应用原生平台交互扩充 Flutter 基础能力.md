<p data-nodeid="673" class="">本课时介绍 Flutter 如何与原生平台进行通信交互方式，让 Flutter 支持各种原生平台的基础能力。</p>
<h3 data-nodeid="674">使用场景</h3>
<p data-nodeid="675">由于 Flutter 是一个跨平台 UI 库，因此不支持原生系统的功能，例如：</p>
<ul data-nodeid="676">
<li data-nodeid="677">
<p data-nodeid="678">系统通知；</p>
</li>
<li data-nodeid="679">
<p data-nodeid="680">系统感应、相机、电量、LBS、声音、语音识别；</p>
</li>
<li data-nodeid="681">
<p data-nodeid="682">分享、打开其他 App 或者打开自身 App；</p>
</li>
<li data-nodeid="683">
<p data-nodeid="684">设备信息、本地存储。</p>
</li>
</ul>
<p data-nodeid="685">以上只列举了部分，其实主要是和系统服务调用相关的功能，大部分都不支持。这时候就需要原生平台提供一些基础服务给 Flutter 来调用。我们先来看下 Flutter 与 Android 和 iOS 是怎么进行消息传递和接收的。</p>
<h3 data-nodeid="686">交互原理</h3>
<p data-nodeid="687">在 Flutter 中存在三种与原生平台进行交互的方法： MethodChannel 、BasicMessageChannel 和 EventChannel 。这三者在底层是没有区别的，都是基于 binaryMessenger 来实现。不过在应用层中的使用场景有所区别。</p>
<ul data-nodeid="688">
<li data-nodeid="689">
<p data-nodeid="690">MethodChannel ，该方法需要创建一个消息通道句柄，然后再利用其中的 invokeMethod 来调用原生平台，原生平台根据传递的方法和参数，执行并获得具体的异步响应结果。该方法支持两个参数，一个是方法名，一个是方法参数，因此更适合去调用原生客户端的函数方法；</p>
</li>
<li data-nodeid="691">
<p data-nodeid="692">BasicMessageChannel ，该方法需要创建一个消息通道句柄，然后再利用其中的 send 方法发送数据给到原生平台.原生平台接收到数据后，可以针对接收数据响应返回，也可以在接收数据后，不做任何返回。因此该方法更适合向原生平台传递数据，而不是功能调用；</p>
</li>
<li data-nodeid="693">
<p data-nodeid="694">EventChannel ，该方法是数据流传递，适用于大文件或者数据流媒体等的应用。发送方不会有响应，但是它会通过调用 MethodChannel 来通知原生平台，比如开始监听数据接收会发送 listen ，取消了数据接收会发送 cancel。</p>
</li>
</ul>
<p data-nodeid="695">在实际应用中三种方法都是有一定场景，大部分情况下还是基于 MethodChannel 来实现，比如前面我们所应用到的插件：FlutterWebviewPlugin 和 PathProviderPlugin ，当然其中也涉及 EventChannel 的应用，比如 UniLinksPlugin 插件。接下来我们具体看下整个消息交互的流转过程。</p>
<h4 data-nodeid="696">交互实现过程</h4>
<p data-nodeid="697">根据官网的知识以及我自己的一个理解，可以将整个过程总结为下图 1 。</p>
<p data-nodeid="698"><img src="https://s0.lgstatic.com/i/image/M00/41/CC/Ciqc1F82QdiASdHvAAGbIVp4bfI196.png" alt="Drawing 0.png" data-nodeid="773"></p>
<div data-nodeid="699"><p style="text-align:center">图 1 消息交互流程图</p></div>
<p data-nodeid="700">从图 1 中我们可以看到，所有的消息都是通过 binaryMessenger 来传递，Flutter 的底层是 C 和 C++ 实现的，binaryMessenger 就是通过 C++ 底层库来调用平台相关的功能，数据返回也是原路处理返回。上面的调用过程，就是 Flutter 官网三层架构（如图 2 所示）的一个典型例子。</p>
<p data-nodeid="701"><img src="https://s0.lgstatic.com/i/image/M00/41/E1/CgqCHl82ToiAUS0SAAGny7Ud86w285.png" alt="image (8).png" data-nodeid="777"></p>
<div data-nodeid="702"><p style="text-align:center">图 2 Flutter 三层架构</p></div>
<h3 data-nodeid="703">应用示例</h3>
<p data-nodeid="704">原理分析清晰后，我们再基于我们当前 Two You Friend App 项目实践一下这个功能。主要需求就是能够在 Flutter 中查看当前电量信息，具体效果如如图 3 所示。</p>
<p data-nodeid="705"><img src="https://s0.lgstatic.com/i/image/M00/41/D6/Ciqc1F82TpCAKE8oAAExi3g6bqk260.png" alt="image (7).png" data-nodeid="782"></p>
<div data-nodeid="706"><p style="text-align:center">图 3 获取电量界面效果图</p></div>
<p data-nodeid="707">从图中我们可以看到在 Android 中是可以正常获取到当前电量信息，但是在 iOS 中是无法获取（主要原因是在虚拟机上 iOS 不支持 device.batteryState 方法）。接下来我们看下具体的代码实现逻辑。</p>
<h4 data-nodeid="708">增加测试页面</h4>
<p data-nodeid="709">在项目中的 lib/pages 下创建一个 test_page 文件夹，在文件夹中创建 index.dart 。因为需要使用到 MethodChannel ，所以在头部增加两个库的引入，代码如下：</p>
<pre class="lang-dart" data-nodeid="710"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/services.dart'</span>;
</code></pre>
<p data-nodeid="711">接下来实现 TestPageIndex 这个类，并在该类中应用 MethodChannel 创建一个消息句柄，代码如下：</p>
<pre class="lang-dart" data-nodeid="712"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">测试页面 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TestPageIndex</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{ 
  <span class="hljs-comment">/// <span class="markdown">构造函数 </span></span>
  <span class="hljs-keyword">const</span> TestPageIndex(); 
  <span class="hljs-comment">/// <span class="markdown">创建数据传输句柄 </span></span>
  <span class="hljs-keyword">static</span> <span class="hljs-keyword">const</span> platform = MethodChannel(<span class="hljs-string">'com.example.two_you_friend'</span>); 
  <span class="hljs-meta">@override</span> 
  Widget build(BuildContext context) { 
    <span class="hljs-keyword">return</span> Container(); <span class="hljs-comment">// @todo</span>
  } 
}
</code></pre>
<p data-nodeid="713">上面代码创建了一个唯一的消息名句柄，为了避免重复性，这里最好的方式就是使用包的名称加上功能。接下来我们实现 build 逻辑。由于调用原生的 invokeMethod 是一个异步消息返回的方法，因此这里需要使用 FutureBuilder<code data-backticks="1" data-nodeid="790">&lt;Widget&gt;</code> 来实现，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="714"><code data-language="dart"><span class="hljs-meta">@override</span> 
Widget build(BuildContext context) { 
  <span class="hljs-keyword">return</span> FutureBuilder&lt;Widget&gt;( 
      future: _getBatteryLevel(), 
      builder: (BuildContext context, AsyncSnapshot&lt;Widget&gt; snapshot) { 
        <span class="hljs-keyword">if</span>(snapshot.error != <span class="hljs-keyword">null</span>) { 
          <span class="hljs-keyword">return</span> Container( 
            child: CommonError(), 
          ); 
        } 
        <span class="hljs-keyword">return</span> Container( 
          child: snapshot.data, 
        ); 
      }); 
}
</code></pre>
<p data-nodeid="715">上面的代码调用了一个异步函数 _getBatteryLevel ，正确返回则显示相应的组件，异常则显示通用报错页面，最后再来看下 _getBatteryLevel 的实现逻辑。</p>
<pre class="lang-dart" data-nodeid="716"><code data-language="dart">Future&lt;Widget&gt; _getBatteryLevel() <span class="hljs-keyword">async</span> { 
  <span class="hljs-built_in">String</span> batteryLevel; 
  <span class="hljs-keyword">try</span> { 
    <span class="hljs-keyword">final</span> <span class="hljs-built_in">int</span> result = <span class="hljs-keyword">await</span> platform.invokeMethod(<span class="hljs-string">'getBatteryLevel'</span>); 
    batteryLevel = <span class="hljs-string">'Battery level at <span class="hljs-subst">$result</span> % .'</span>; 
    <span class="hljs-built_in">print</span>(batteryLevel); 
  } <span class="hljs-keyword">on</span> PlatformException <span class="hljs-keyword">catch</span> (e) { 
    <span class="hljs-built_in">print</span>(e.message); 
    batteryLevel = <span class="hljs-string">"Failed to get battery level: '<span class="hljs-subst">${e.message}</span>'."</span>; 
  } 
  <span class="hljs-keyword">return</span> Center( 
    child: Text(batteryLevel), 
  ); 
}
</code></pre>
<p data-nodeid="717">创建一个空字符串，然后通过 platform.invokeMethod 发送给原生平台，原生平台异步返回消息得到 result 结果，由于 invokeMethod 是一个范型，因此可以将结果设置为 int 类型。这里使用 try catch 的目的是避免因为原生平台不支持导致的异常，比如 iOS 就不支持在虚拟机上调用该 API 。</p>
<h4 data-nodeid="718">增加页面跳转</h4>
<p data-nodeid="719">页面实现完成后，我们再去 router 中增加该页面的配置。首先使用 import 引入该页面，然后再修改 lib/router.dart 在 routerMapping 中增加一项。</p>
<pre class="lang-dart" data-nodeid="720"><code data-language="dart"><span class="hljs-string">'test'</span>: StructRouter(TestPageIndex(), <span class="hljs-number">-1</span>, <span class="hljs-keyword">null</span>, <span class="hljs-keyword">true</span>),
</code></pre>
<p data-nodeid="721">完成路由配置后，再前往 lib/widgets/menu/draw.dart 文件，在 ListView 下的 children 中增加一个新的菜单，代码配置如下：</p>
<pre class="lang-dart" data-nodeid="722"><code data-language="dart">ListTile( 
  leading: Icon(Icons.person), 
  title: Text(<span class="hljs-string">'原生测试'</span>), 
  onTap: () { 
    Navigator.pop(context); 
    redirect(<span class="hljs-string">'tyfapp://test'</span>); 
  }, 
),
</code></pre>
<p data-nodeid="723">以上就完成了在 Flutter 中的代码逻辑。运行程序后，我们是可以正常打开该页面的，只是没有正确的响应数据，接下来我们就分平台实现获取平台电量的代码。</p>
<h4 data-nodeid="724">Android 代码</h4>
<p data-nodeid="725">在项目根目录，我们有一个 android 的目录文件夹，使用 Android Studio 打开该项目，然后在 app/java 下创建一个 com.example.two_you_friend 这样的包名（在 Android Studio 是叫作 Package ），如果你不想再打开一个项目，也可以在当前项目的 android/app/src/main/java 目录下创建 com.example.two_you_friend 包，然后在该目录下新建一个 MainActivity.java 文件。接下来我们看下具体的代码实现。</p>
<p data-nodeid="726">第一步还是 import 我们需要的库文件。</p>
<pre class="lang-java" data-nodeid="727"><code data-language="java"><span class="hljs-keyword">package</span> com.example.two_you_friend; 
<span class="hljs-keyword">import</span> android.content.ContextWrapper; 
<span class="hljs-keyword">import</span> android.content.Intent; 
<span class="hljs-keyword">import</span> android.content.IntentFilter; 
<span class="hljs-keyword">import</span> android.os.BatteryManager; 
<span class="hljs-keyword">import</span> android.os.Build.VERSION; 
<span class="hljs-keyword">import</span> android.os.Build.VERSION_CODES; 
<span class="hljs-keyword">import</span> androidx.annotation.NonNull; 
<span class="hljs-keyword">import</span> io.flutter.embedding.android.FlutterActivity; 
<span class="hljs-keyword">import</span> io.flutter.embedding.engine.FlutterEngine; 
<span class="hljs-keyword">import</span> io.flutter.plugin.common.MethodChannel; 
<span class="hljs-keyword">import</span> io.flutter.plugins.GeneratedPluginRegistrant;
</code></pre>
<p data-nodeid="728">接下来创建 MainActivity 来继承 FlutterActivity ，创建一个与 Flutter 中对应的消息名字，并创建两个未实现的方法 configureFlutterEngine 和 getBatteryLevel 。</p>
<pre class="lang-java" data-nodeid="729"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MainActivity</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">FlutterActivity</span> </span>{ 
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String CHANNEL = <span class="hljs-string">"com.example.two_you_friend"</span>; 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configureFlutterEngine</span><span class="hljs-params">(<span class="hljs-meta">@NonNull</span> FlutterEngine flutterEngine)</span> </span>{ 
    } 

    <span class="hljs-comment">/** 
     * 获取电量信息 
     * <span class="hljs-doctag">@return</span> string 
     */</span> 
    <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> <span class="hljs-title">getBatteryLevel</span><span class="hljs-params">()</span> </span>{ 
    } 
}
</code></pre>
<p data-nodeid="730">configureFlutterEngine 重写的父类方法，主要是用来处理 MethodChannel 发送过来的数据。getBatteryLevel 获取当前电量信息。我们先来看下 configureFlutterEngine 代码：</p>
<pre class="lang-dart" data-nodeid="731"><code data-language="dart"><span class="hljs-meta">@Override</span> 
public <span class="hljs-keyword">void</span> configureFlutterEngine(<span class="hljs-meta">@NonNull</span> FlutterEngine flutterEngine) { 
    GeneratedPluginRegistrant.registerWith(flutterEngine); 
    <span class="hljs-keyword">new</span> MethodChannel(flutterEngine.getDartExecutor().getBinaryMessenger(), CHANNEL) 
            .setMethodCallHandler( 
                    (call, result) -&gt; { 
                        <span class="hljs-keyword">if</span> (call.method.equals(<span class="hljs-string">"getBatteryLevel"</span>)) { 
                            <span class="hljs-built_in">int</span> batteryLevel = getBatteryLevel(); 
                            <span class="hljs-keyword">if</span> (batteryLevel != <span class="hljs-number">-1</span>) { 
                                result.success(batteryLevel); 
                            } <span class="hljs-keyword">else</span> { 
                                result.error(<span class="hljs-string">"UNAVAILABLE"</span>, <span class="hljs-string">"Battery level not available."</span>, <span class="hljs-keyword">null</span>); 
                            } 
                        } <span class="hljs-keyword">else</span> { 
                            result.notImplemented(); 
                        } 
                    } 
            ); 
}
</code></pre>
<p data-nodeid="732">上面代码核心部分是在 call.method.equals ，判断 Flutter 需要调用的方法，根据不同的数据调用不同的函数，比如 getBatteryLevel 则调用 getBatteryLevel() 。最后我们再来看下电量获取部分的代码：</p>
<pre class="lang-java" data-nodeid="733"><code data-language="java"><span class="hljs-comment">/** 
 * 获取电量信息 
 * <span class="hljs-doctag">@return</span> string 
 */</span> 
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> <span class="hljs-title">getBatteryLevel</span><span class="hljs-params">()</span> </span>{ 
    <span class="hljs-keyword">int</span> batteryLevel = -<span class="hljs-number">1</span>; 
    <span class="hljs-keyword">if</span> (VERSION.SDK_INT &gt;= VERSION_CODES.LOLLIPOP) { 
        BatteryManager batteryManager = (BatteryManager) getSystemService(BATTERY_SERVICE); 
        batteryLevel = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY); 
    } <span class="hljs-keyword">else</span> { 
        Intent intent = <span class="hljs-keyword">new</span> ContextWrapper(getApplicationContext()). 
                registerReceiver(<span class="hljs-keyword">null</span>, <span class="hljs-keyword">new</span> IntentFilter(Intent.ACTION_BATTERY_CHANGED)); 
        batteryLevel = (intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -<span class="hljs-number">1</span>) * <span class="hljs-number">100</span>) / 
                intent.getIntExtra(BatteryManager.EXTRA_SCALE, -<span class="hljs-number">1</span>); 
    } 
    <span class="hljs-keyword">return</span> batteryLevel; 
}
</code></pre>
<p data-nodeid="734">由于是 Java 代码，我了解的也较少，也非本课时的知识点，具体大家参考下了解就可以。<br>
以上就完成了整个开发过程，不过这里有一个非常大的坑，在 Flutter 项目创建完成后，app 目录下的 src/main/kotlin 目录中也会存在另外一个 MainActivity 类，这样会导致 Android 项目编译失败，可以将 src/main/kotlin 下的 MainActivity 类改一个名字即可，需要时再将 java 和 kotlin 中的两个类名修改回来。</p>
<p data-nodeid="735">开发完成后，就可以使用 Android 虚拟机来测试了。接下来我们看下 iOS 代码。</p>
<h4 data-nodeid="736">iOS 代码</h4>
<p data-nodeid="737">iOS 也支持两种语言 Object-C 和 Swift ，这里我们使用 Swift 来演示。直接在 Android Studio 中打开 ios/Runner 目录下的 .swift 文件。添加下以下部分代码，由于是 Swift 代码，我就不过多介绍如何实现的细节了。</p>
<pre class="lang-swift" data-nodeid="738"><code data-language="swift"><span class="hljs-keyword">import</span> UIKit 
<span class="hljs-keyword">import</span> Flutter 
<span class="hljs-meta">@UIApplicationMain</span> 
<span class="hljs-meta">@objc</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AppDelegate</span>: <span class="hljs-title">FlutterAppDelegate</span> </span>{ 
 <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">application</span><span class="hljs-params">( 
  <span class="hljs-number">_</span> application: UIApplication, 
  didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: <span class="hljs-keyword">Any</span>]?)</span></span> -&gt; <span class="hljs-type">Bool</span> { 
  <span class="hljs-keyword">let</span> controller : <span class="hljs-type">FlutterViewController</span> = window?.rootViewController <span class="hljs-keyword">as</span>! <span class="hljs-type">FlutterViewController</span> 
  <span class="hljs-keyword">let</span> batteryChannel = <span class="hljs-type">FlutterMethodChannel</span>(name: <span class="hljs-string">"com.example.two_you_friend"</span>, 
                       binaryMessenger: controller.binaryMessenger) 
  batteryChannel.setMethodCallHandler({ 
   [<span class="hljs-keyword">weak</span> <span class="hljs-keyword">self</span>] (call: <span class="hljs-type">FlutterMethodCall</span>, result: <span class="hljs-type">FlutterResult</span>) -&gt; <span class="hljs-type">Void</span> <span class="hljs-keyword">in</span> 
   <span class="hljs-comment">// Note: this method is invoked on the UI thread. </span>
   <span class="hljs-keyword">guard</span> call.method == <span class="hljs-string">"getBatteryLevel"</span> <span class="hljs-keyword">else</span> { 
    result(<span class="hljs-type">FlutterMethodNotImplemented</span>) 
    <span class="hljs-keyword">return</span> 
   } 
   <span class="hljs-keyword">self</span>?.receiveBatteryLevel(result: result) 
  }) 
  <span class="hljs-type">GeneratedPluginRegistrant</span>.register(with: <span class="hljs-keyword">self</span>) 
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">super</span>.application(application, didFinishLaunchingWithOptions: launchOptions) 
 } 
  <span class="hljs-keyword">private</span> <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">receiveBatteryLevel</span><span class="hljs-params">(result: FlutterResult)</span></span> { 
   <span class="hljs-keyword">let</span> device = <span class="hljs-type">UIDevice</span>.current 
   device.isBatteryMonitoringEnabled = <span class="hljs-literal">true</span> 
   <span class="hljs-keyword">if</span> device.batteryState == <span class="hljs-type">UIDevice</span>.<span class="hljs-type">BatteryState</span>.unknown { 
    result(<span class="hljs-type">FlutterError</span>(code: <span class="hljs-string">"UNAVAILABLE"</span>, 
              message: <span class="hljs-string">"Battery info unavailable"</span>, 
              details: <span class="hljs-literal">nil</span>)) 
   } <span class="hljs-keyword">else</span> { 
    result(<span class="hljs-type">Int</span>(device.batteryLevel * <span class="hljs-number">100</span>)) 
   } 
  } 
}
</code></pre>
<p data-nodeid="739">代码完成后，使用 iOS 模拟器运行项目就可以看到图 3 所示的一个效果了。</p>
<h3 data-nodeid="740">Flutter 插件</h3>
<p data-nodeid="741">学习掌握与原生通信原理后，我们就可以利用该功能做一些跨平台原生 Flutter 插件，通过插件的方式来屏蔽平台特性。具体大家可以尝试创建一个试试：</p>
<p data-nodeid="742">1.在 Android Studio 创建一个新的项目，项目类型选择 Flutter Plugin ，或者使用下面的命令行；</p>
<pre class="lang-powershell" data-nodeid="743"><code data-language="powershell">flutter create -<span class="hljs-literal">-org</span> com.example -<span class="hljs-literal">-template</span>=plugin plugin_name
</code></pre>
<p data-nodeid="744">2.创建完成后，里面会包含相应的测试代码，以及会准备好最基础的代码部分，只需要在模版代码上增加我们应用示例的代码；</p>
<p data-nodeid="745">3.创建完成后，在不修改示例的基础上运行，可以看到如图 4 所示的一个效果。</p>
<p data-nodeid="746"><img src="https://s0.lgstatic.com/i/image/M00/41/E1/CgqCHl82TqCAWzXCAAAwHz1zoOw595.png" alt="image (9).png" data-nodeid="830"></p>
<div data-nodeid="747"><p style="text-align:center">图 4 Flutter Plugin 效果</p></div>
<p data-nodeid="748">4.开发完成插件后，如果需要使用该插件，方法还是在 pubspec.yaml 增加依赖，例如下面的配置。</p>
<pre class="lang-yaml" data-nodeid="749"><code data-language="yaml"><span class="hljs-attr">dependencies:</span> 
  <span class="hljs-attr">flutter:</span> 
    <span class="hljs-attr">sdk:</span> <span class="hljs-string">flutter</span> 
  <span class="hljs-attr">battery:</span> 
    <span class="hljs-comment"># When depending on this package from a real application you should use: </span>
    <span class="hljs-comment">#   battery2: ^x.y.z </span>
    <span class="hljs-comment"># See https://dart.dev/tools/pub/dependencies#version-constraints </span>
    <span class="hljs-comment"># The example app is bundled with the plugin so we use a path dependency on </span>
    <span class="hljs-comment"># the parent directory to use the current plugin's version.</span>
    <span class="hljs-attr">path:</span> <span class="hljs-string">../</span>
</code></pre>
<p data-nodeid="750">battery 是我们测试的插件名称，path 是一个相对路径，指向插件的项目根目录即可。</p>
<p data-nodeid="751">以上就是原生插件的开发过程，需要有一定的原生平台开发能力，这也是我一开始介绍到的后面大前端的一个方向，跨端团队作为业务支撑，而 Android 、 iOS 以及 Web 作为平台底层支持跨端的团队。</p>
<h3 data-nodeid="752">总结</h3>
<p data-nodeid="753">本课时核心是介绍了如何在 Flutter 中与原生平台进行通信，从而扩充 Flutter 基础功能，这部分还是需要有一定的原生编程能力。在掌握通信机制后，也顺便介绍了如何创建 Flutter plugin ，从而将多平台代码作为插件进行开发，而在 Flutter 端屏蔽多终端的问题。学完本课时以后，需要掌握 Flutter 与原生平台的通信方式，并且了解 Flutter plugin 的开发过程。</p>
<p data-nodeid="1333">下一课时我将介绍 Flutter 中的性能监控和分析，并利用性能分析来优化我们当前 Two You APP 项目。</p>
<p data-nodeid="1669" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="1672">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### **炎：
> 老师，flutter做直播APP推荐吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不推荐做直播的核心业务，因为 flutter 根本做不到，主要还是音视频处理还是得依赖原生，但是可以结合原来一起做，比如一些非音视频处理的业务。

##### **远：
> 这个课程实用

