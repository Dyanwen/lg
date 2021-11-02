<p data-nodeid="42225" class="">本课时主要带你学习 Flutter 中的性能分析工具 —— devTools 的使用，以及掌握 Flutter 性能分析思路和方法。其次也会介绍到在 Flutter 中如何上报性能相关的数据，从服务端数据出发分析 App 用户体验数据。</p>
<h3 data-nodeid="42226">devTools 的使用</h3>
<p data-nodeid="42227">devTools 是官方出的一套 Dart 和 Flutter 的性能调试工具，其核心是帮开发者定位 UI 或者 GPU 线程问题，从而协助开发者解决 Flutter App 的性能问题。在应用该工具之前，需要启动 App 调试功能。</p>
<p data-nodeid="42228">打开我们的 Two You App，如果是在 Android Studio IDE 中，可以直接点击如下图所示的红色圈部分。</p>
<p data-nodeid="42229"><img src="https://s0.lgstatic.com/i/image/M00/43/EA/Ciqc1F886IyANVraAABwZCwQP1s839.png" alt="Drawing 0.png" data-nodeid="42358"></p>
<div data-nodeid="42230"><p style="text-align:center">打开 devTools 指引</p></div>
<p data-nodeid="42231">如果不是在 Android Studio 中，需要按照下面的四个步骤启动 devTools 工具。</p>
<p data-nodeid="42232">1.使用下面命令启动 devTools 工具。</p>
<pre class="lang-plain" data-nodeid="42233"><code data-language="plain">flutter pub global run devtools
</code></pre>
<p data-nodeid="42234">2.运行成功后，会提示 devTools 访问的地址。打开访问地址后，可以看到如下图的界面，界面中需要输入一个 Flutter App 的监听地址。</p>
<p data-nodeid="42235"><img src="https://s0.lgstatic.com/i/image/M00/43/F5/CgqCHl886JiAFj7bAABf4KvYysQ626.png" alt="Drawing 1.png" data-nodeid="42364"></p>
<div data-nodeid="42236"><p style="text-align:center">devTools 主界面</p></div>
<p data-nodeid="42237">3.接下来需要获取 Flutter 运行的 WS 地址，重新运行项目（请注意不是热启动，需要停止运行，然后点击重新运行），启动成功后，在运行栏可以看到如下图所示的信息。</p>
<p data-nodeid="42238"><img src="https://s0.lgstatic.com/i/image/M00/43/EA/Ciqc1F886KuAF95mAACDP4U2o5I743.png" alt="Drawing 2.png" data-nodeid="42368"></p>
<div data-nodeid="42239"><p style="text-align:center">调试提示</p></div>
<p data-nodeid="42240">4.将其中的 listening on 的地址输入到刚才 devTools 页面就可以了，打开页面后，可以看到下图所示的功能。</p>
<p data-nodeid="42241"><img src="https://s0.lgstatic.com/i/image/M00/43/F5/CgqCHl886LeAHz3YAAEP13-CfMA452.png" alt="Drawing 3.png" data-nodeid="42372"></p>
<div data-nodeid="42242"><p style="text-align:center">devTools 功能</p></div>
<p data-nodeid="42243">接下来我介绍下 devTools 功能，即上图中每个工具的作用。</p>
<ul data-nodeid="42244">
<li data-nodeid="42245">
<p data-nodeid="42246"><strong data-nodeid="42378">Flutter Inspector</strong>，可以查看组件的布局信息，类似于前端的 Chrome 工具的 CSS 布局查看器，应用该功能可以快速定位布局问题。</p>
</li>
<li data-nodeid="42247">
<p data-nodeid="42248"><strong data-nodeid="42383">Timeline</strong>，时间线事件图表，跟踪显示来自应用程序的所有事件。监听 Flutter App 在构建 UI 树，绘制界面以及其他（例如 HTTP 流量）事件等，并将监听到的事件所花费的时间，显示在时间轴上。</p>
</li>
<li data-nodeid="42249">
<p data-nodeid="42250"><strong data-nodeid="42388">Memory</strong>，使用时间线的方式，展示 Flutter App 的内存变化，通过该工具可以定位内存泄漏的问题。</p>
</li>
<li data-nodeid="42251">
<p data-nodeid="42252"><strong data-nodeid="42393">Performance</strong>，性能分析工具，可以通过录制界面操作，获取界面性能数据。该工具的主要用途还是在定位某个功能卡顿问题，例如我们发现主界面很卡顿，这时候就可以通过该工具录制首页加载过程，然后分析出具体性能异常逻辑。</p>
</li>
<li data-nodeid="42253">
<p data-nodeid="42254"><strong data-nodeid="42398">Debugger</strong>，断点调试功能，和 IDE 上的断点调试是一样的。</p>
</li>
<li data-nodeid="42255">
<p data-nodeid="42256"><strong data-nodeid="42403">Network</strong>，可以抓取网络请求，并分析返回数据，类似于前端 Chrome 的 Network 工具。</p>
</li>
<li data-nodeid="42257">
<p data-nodeid="42258"><strong data-nodeid="42408">Logging</strong>，运行期间的日志显示。日志中包含：Dart 运行时的垃圾回收事件、Flutter 框架事件，比如创建帧的事件、应用的 stdout 和 stderr 和应用的自定义日志事件。</p>
</li>
</ul>
<p data-nodeid="42259">介绍完以上功能后，我们着重介绍 Timeline 工具，应用该工具来做性能分析和优化。</p>
<h4 data-nodeid="42260">Timeline</h4>
<p data-nodeid="42261">Timeline 会记录每一帧的绘制，每一帧绘制又包括 UI 线程构建图形树和 GPU 线程绘制图像两个过程。在应用开发完成后，我们可以使用 Timeline 工具来走一遍 App 所有页面，记录每一帧的性能耗时。请注意该功能最好是在外接实体机上进行测试，不然会出现数据相差较大。我们分为以下七个步骤来进行实践。</p>
<p data-nodeid="42262">1.连接实体调试机器，然后运行 flutter run --profile 启动 App。<br>
2.打开 devTools 工具，点击 Timeline 工具，点击 Clear 清空旧数据。<br>
3.可以在某个页面上进行一系列的基础操作，操作完成后，回到 devTools 中，点击 Refresh，这时候会有一个短暂的分析过程，分析完成后，你会看到下图所示的内容。</p>
<p data-nodeid="42263"><img src="https://s0.lgstatic.com/i/image/M00/43/EA/Ciqc1F886MKAXHUJAADEduC9Pvw266.png" alt="Drawing 4.png" data-nodeid="42419"></p>
<div data-nodeid="42264"><p style="text-align:center">Timeline 性能柱状图</p></div>
<p data-nodeid="46046" class="">4.在界面中，你会看到浅蓝（ UI 线程耗时，小于  16.67ms）、深蓝（ GPU 耗时，小于 16.67 ms）、橘黄（ UI 线程耗时，大于 16.67 ms）和深红（ GPU 耗时，大于 16.67 ms）的柱状数据，浅蓝和橘黄都代表 UI 线程耗时，深蓝和深红都代表 GPU 耗时，在 UI 线程耗时和 GPU 耗时都小于 16.67 ms 时显示浅蓝和深蓝，而当 UI 线程耗时或者 GPU 耗时任意一个大于 16.67 ms 时，则显示橘黄和深红。</p>







<p data-nodeid="42266">5.当发现有橘黄和深红的柱状图时，则需要进行具体的分析，这时候只需要点击这部分柱状图，就可以看到下图所示的一个效果。</p>
<p data-nodeid="42267"><img src="https://s0.lgstatic.com/i/image/M00/43/EA/Ciqc1F886NuAOv4lAAMBs3hz5KY064.png" alt="Drawing 5.png" data-nodeid="42424"></p>
<div data-nodeid="42268"><p style="text-align:center">Timeline 单个数据分析图</p></div>
<p data-nodeid="42269">6.如果 UI 线程耗时比较长，点击具体较长的柱状图，可以看到具体的火焰图。如下图所示，其中的宽度就代表执行的时间长度，宽度越长表明性能损耗越大，而这就是性能优化的部分。</p>
<p data-nodeid="42270"><img src="https://s0.lgstatic.com/i/image/M00/43/EA/Ciqc1F886OOABOHtAAOrn9ZoZMo713.png" alt="Drawing 6.png" data-nodeid="42428"></p>
<div data-nodeid="42271"><p style="text-align:center">UI 线程耗时分析</p></div>
<p data-nodeid="42272">7.如果 GPU 耗时较长，则可以往下拉查看 GPU 页面绘制问题，如下图所示。</p>
<p data-nodeid="42273"><img src="https://s0.lgstatic.com/i/image/M00/43/F5/CgqCHl886OqAdQZ0AAJb7VYSQsA920.png" alt="Drawing 7.png" data-nodeid="42432"></p>
<div data-nodeid="42274"><p style="text-align:center">GPU 耗时分析</p></div>
<p data-nodeid="42275">接下来我们就分别从 UI 线程问题和 GPU （ Raster ）来分析具体的性能问题。</p>
<h4 data-nodeid="42276">UI 线程问题实践分析</h4>
<p data-nodeid="42277">如果出现橘黄色和深红色的柱状图时，我们需要单独分析这块的性能问题。大部分情况是因为在 Dart 中执行了比较耗时的函数，或者在组件树设计上没有注意性能导致的问题。这里介绍下可能会提升或者影响性能的几个关键点。</p>
<ul data-nodeid="42278">
<li data-nodeid="42279">
<p data-nodeid="42280">不会发生任何变化的组件，使用 const ，减少绘制，例如我们的通用 loading 组件。</p>
</li>
<li data-nodeid="42281">
<p data-nodeid="42282">减少组件绘制，这点就是我们之前提到的尽量减少有状态组件下的子组件，或者通过状态管理模块 Provider 来辅助管理状态。</p>
</li>
<li data-nodeid="42283">
<p data-nodeid="42284">复杂业务 build 函数在代码逻辑中，避免复杂业务在 build 逻辑中去执行。例如我们代码中的 user_page/guest.dart 这个页面，这个页面逻辑是相当复杂的，首先需要使用 JSON 的方式获取参数，其次还需要进行接口拉取，这块是比较耗费性能的。</p>
</li>
</ul>
<p data-nodeid="42285">例如这里我们需要分析 guest.dart 页面的问题。首先我们连接真机调试，然后启动程序，可以看到如下图的界面，并选择 Track Widget Builds 统计（该工具会显示具体的组件名字，以方便我们查看问题）。</p>
<p data-nodeid="42286"><img src="https://s0.lgstatic.com/i/image/M00/43/F5/CgqCHl886PKAA9UrAAFt4lddgwk448.png" alt="Drawing 8.png" data-nodeid="42444"></p>
<div data-nodeid="42287"><p style="text-align:center">Timeline 调试主页</p></div>
<p data-nodeid="42288">接下来在真机上，点击某个人的头像，进入用户页面，也就是访问我们的 user_page/guest.dart 。访问完成后，点击 Timeline 工具中的 Refresh ，就可以看到如下图的界面。</p>
<p data-nodeid="42289"><img src="https://s0.lgstatic.com/i/image/M00/43/F5/CgqCHl886PiAccZGAADxVxH-wTs542.png" alt="Drawing 9.png" data-nodeid="42450"></p>
<div data-nodeid="42290"><p style="text-align:center">Refresh 性能检测结果</p></div>
<p data-nodeid="42291">从上面，我们可以很明显地看到几条橘黄色的柱状图，我们随便选择一个去分析，点击后，可以在界面中看到下图的 UI 构建耗时分布的火焰图。</p>
<p data-nodeid="42292"><img src="https://s0.lgstatic.com/i/image/M00/43/EA/Ciqc1F886QKAFvlNAAHmvu171ZY675.png" alt="Drawing 10.png" data-nodeid="42454"></p>
<div data-nodeid="42293"><p style="text-align:center">UI 耗时火焰图</p></div>
<p data-nodeid="42294">接下来我们就可以看到 build 逻辑调用的各个组件的耗时情况，也可以看到这其中构建了多少个组件，其次通过这张图也可以快速地定位到哪些组件没有更新。在上面图中，我们可以看到 BottomNavigationBar 相对其他组件来说，耗时就非常大。那我们就具体来看看，到底是什么原因导致的。</p>
<p data-nodeid="42295">我们在 guest.dart 中并没有应用 BottomNavigationBar 组件，那么怎么会产生调用呢？接下来我们就使用 debug 功能，在 guest.dart 的 build 逻辑加上断点，然后使用 debug 模式运行。</p>
<p data-nodeid="42296">这个问题最终发现是因为 Navigator.push 引发 Scaffold 的更新，而 Scaffold 是 BottomNavigationBar 和 AppBar 的父组件，从而又引发了这个组件的下的子组件发生 rebuild 行为。之前官方有修复一个 bug 就是在 Navigator.push 会引起前面的页面 rebuild 操作，但是这个问题好像没有发现并引起重视。关于这个问题我已经提交 <a href="https://github.com/flutter/flutter/issues/63312" data-nodeid="42460">github issue</a> ，如果有进展或者你有解决方案，也帮忙在此评论下。</p>
<p data-nodeid="42297">其他问题寻找方式也是类似的，只要按照上面的过程去分析。</p>
<h4 data-nodeid="42298">GPU（ Raster ）</h4>
<p data-nodeid="42299">一般情况下都是较复杂的图片绘制产生的问题，比如说复杂的动效或者复杂的图片资源。上面的工具也不能完全帮你定位到异常的问题。需要根据实际的代码逻辑来分析，这点是比较困难的，只能排除法步步寻找问题点。Timeline 图只能协助我们去找到 GPU 存在性问题。</p>
<p data-nodeid="42300">你在遇到 GPU 问题时，可以在 devTools 中的 Timeline 打开 Performance Overlay 工具，打开后在真机或者虚拟机上就可以看到效果，当出现 GPU 性能问题时，会出现红色线条。</p>
<p data-nodeid="42301"><img src="https://s0.lgstatic.com/i/image/M00/43/F5/CgqCHl886Q6AUmANAAEEvCCE5Hg305.png" alt="Drawing 11.png" data-nodeid="42468"></p>
<div data-nodeid="42302"><p style="text-align:center">Performance Overlay</p></div>
<p data-nodeid="42303">以上就是 devTools 的工具使用，通过这个工具可以大大提升我们定位问题的效率。以上是在开发中或者上线前所需要做的一些前期性能分析准备，那么在上线后我们应该如何来分析性能呢？</p>
<h3 data-nodeid="42304">性能上报</h3>
<p data-nodeid="42305">为了能够更好地分析和判断性能问题，我们有时候需要采集现网运行期间的一些性能数据，例如我们需要主要的两个指标：Crash 率和 FPS 数据。接下来我们主要介绍下如何计算和采集这两个数据的方法。</p>
<p data-nodeid="42306">由于这部分肯定会影响主线程的性能，因此我们将该计算和上报过程放入到一个新的线程去处理，避免影响主线程。</p>
<h4 data-nodeid="42307">Isolate 线程</h4>
<p data-nodeid="42308">这部分知识点，在之前的课时中已经有介绍到了，这部分代码在 lib/util/tools/isolate_handle.dart 中。代码中唯一的不同点是需要进行双向的通信，该原理的实现思想是：在新线程回调函数中，向主线程发送一个回包，回包的内容就是自身接收信息句柄，然后主线程可以应用该句柄发送消息给到新线程。主线程代码如下：</p>
<pre class="lang-dart" data-nodeid="42309"><code data-language="dart"><span class="hljs-keyword">var</span> receivePort = ReceivePort(); 
<span class="hljs-keyword">if</span>(isolate == <span class="hljs-keyword">null</span> ) { 
  isolate = <span class="hljs-keyword">await</span> Isolate.spawn(otherIsolate, receivePort.sendPort); 
} 
<span class="hljs-built_in">Map</span> dataInfo = { 
  <span class="hljs-string">'fun'</span> : callFun, 
  <span class="hljs-string">'routerFrames'</span> : routerFrames, 
  <span class="hljs-string">'deviceInfo'</span> : deviceInfo, 
  <span class="hljs-string">'routerName'</span> : routerName 
}; 
<span class="hljs-keyword">if</span>(sendPort != <span class="hljs-keyword">null</span>){ <span class="hljs-comment">// 如果已经连接成功，则直接使用 sendPort 发送消息 </span>
  sendPort.send(dataInfo); 
} 
receivePort.listen((data) { 
  <span class="hljs-keyword">if</span> (data <span class="hljs-keyword">is</span> SendPort) { <span class="hljs-comment">// 创建初始连接 </span>
    sendPort = data; 
    <span class="hljs-keyword">return</span>; 
  } 
  sendPort.send(dataInfo); 
});
</code></pre>
<p data-nodeid="42310">为了避免创建太多的线程，我们需要将线程连接保存起来，下次可以直接发送消息。再看下新线程的处理逻辑。</p>
<pre class="lang-dart" data-nodeid="42311"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">其他线程，处理计算和上报 </span></span>
<span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> otherIsolate(SendPort sendPort) <span class="hljs-keyword">async</span> { 
  <span class="hljs-keyword">var</span> receivePort = ReceivePort(); 
  receivePort.listen((data) { 
    <span class="hljs-keyword">if</span>(data[<span class="hljs-string">'fun'</span>] == <span class="hljs-string">'calculateFps'</span>) { 
      isolateCalculateFps( 
          data[<span class="hljs-string">'routerFrames'</span>] <span class="hljs-keyword">as</span> ListQueue&lt;FrameTiming&gt;, 
          data[<span class="hljs-string">'routerName'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">String</span>, 
          data[<span class="hljs-string">'deviceInfo'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">String</span> 
      ); 
    } 
    <span class="hljs-keyword">if</span>(data[<span class="hljs-string">'fun'</span>] == <span class="hljs-string">'reportPv'</span>) { 
      isolateReportPv( 
          data[<span class="hljs-string">'routerName'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">String</span>, 
          data[<span class="hljs-string">'deviceInfo'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">String</span> 
      ); 
    } 
  }); 
  <span class="hljs-comment">// 首先需要回句柄，创建通信连接 </span>
  sendPort.send(receivePort.sendPort); 
  <span class="hljs-comment">// 再发送回包，处理具体的信息 </span>
  sendPort.send(<span class="hljs-string">'success'</span>); 
}
</code></pre>
<p data-nodeid="42312">代码的第 22 行首先回第一个包执行连接成功，然后再发送一个回包，用来告诉主线程已经连接成功。连接成功后就可以发送具体的消息实体，让新线程来处理了。以上就是双向通信的实现，<a href="https://github.com/love-flutter/flutter-column" data-nodeid="42481">详细代码大家请参考 Github 源码</a>。接下来我们看下两个指标的实现逻辑。</p>
<h4 data-nodeid="42313">Crash 率</h4>
<p data-nodeid="42314">异常率的计算方法是需要根据手机机型和手机版本来进行分析，我们先制定如下数据指标：</p>
<ul data-nodeid="42315">
<li data-nodeid="42316">
<p data-nodeid="42317">机型的 Crash 率 = 机型的 Crash 量 / 该机型页面访问量</p>
</li>
<li data-nodeid="42318">
<p data-nodeid="42319">版本的 Crash 率 = 版本的 Crash 量 / 该版本页面的访问量</p>
</li>
<li data-nodeid="42320">
<p data-nodeid="42321">版本机型的 Crash 率 = 机型版本的 Crash 量 / 该机型特定版本的访问量</p>
</li>
</ul>
<p data-nodeid="42322">根据上面的计算方式，我们需要增加一些数据上报，主要包括：机型、版本、页面名称、Crash 情况。之前我们已经介绍了 Sentry 平台上报异常的方法，这里只需要补充上页面的 PV 就可以了。</p>
<p data-nodeid="42323">在上面的计算公式中，需要获取设备和版本信息，这部分可以使用通用类来实现，这里我们将代码临时也放在 IsolateHandle 类中，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="42324"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取设备信息 </span></span>
<span class="hljs-keyword">static</span> Future&lt;<span class="hljs-built_in">String</span>&gt; getDeviceInfo() <span class="hljs-keyword">async</span>{ 
  <span class="hljs-built_in">String</span> deviceName = <span class="hljs-string">''</span>; 
  <span class="hljs-comment">/// <span class="markdown">获取设备插件句柄 </span></span>
  DeviceInfoPlugin deviceInfo = DeviceInfoPlugin(); 
  <span class="hljs-keyword">if</span>(Platform.isAndroid) { <span class="hljs-comment">// 判断平台 </span>
    AndroidDeviceInfo androidInfo = <span class="hljs-keyword">await</span> deviceInfo.androidInfo; 
    deviceName = androidInfo.model; 
  } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (Platform.isIOS) { <span class="hljs-comment">// 判断平台 </span>
    IosDeviceInfo iosInfo = <span class="hljs-keyword">await</span> deviceInfo.iosInfo; 
    deviceName = iosInfo.utsname.machine; 
  } 
  <span class="hljs-comment">// 获取当前客户端版本信息 </span>
  PackageInfo packageInfo = <span class="hljs-keyword">await</span> PackageInfo.fromPlatform(); 
  <span class="hljs-built_in">String</span> version = packageInfo.version; 
  <span class="hljs-keyword">return</span> <span class="hljs-string">'<span class="hljs-subst">${deviceName}</span>\t<span class="hljs-subst">${version}</span>'</span>; 
}
</code></pre>
<p data-nodeid="42325">上面代码中应用到了两个第三方库，package_info 获取版本相关信息，device_info 获取设备相关信息。</p>
<pre class="lang-dart" data-nodeid="42326"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:package_info/package_info.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:device_info/device_info.dart'</span>;
</code></pre>
<p data-nodeid="42327">获取到设备后，我们只需要在每次打开页面时进行数据上报就可以了，代码如下：</p>
<pre class="lang-dart" data-nodeid="42328"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">这里上报 pv 数据 </span></span>
<span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> isolateReportPv(<span class="hljs-built_in">String</span> routerName, <span class="hljs-built_in">String</span> deviceInfo) <span class="hljs-keyword">async</span> { 
  <span class="hljs-built_in">print</span>(<span class="hljs-string">'<span class="hljs-subst">${deviceInfo}</span>\t<span class="hljs-subst">${routerName}</span>'</span>); 
  <span class="hljs-comment">/// <span class="markdown">@todo report to server </span></span>
}
</code></pre>
<h4 data-nodeid="42329">FPS</h4>
<p data-nodeid="42330">计算 FPS 的逻辑相对来说较复杂一些，首先需要使用 Flutter 的 SchedulerBinding.instance.addTimingsCallback 函数来获取每一帧耗时，这段代码主要是在 Flutter 绘制完成每一帧后都会进行回调处理，通过回调的方式可以采集到每一帧的耗时信息，具体代码逻辑如下：</p>
<pre class="lang-dart" data-nodeid="42331"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">启动监听数据 </span></span>
<span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> start() <span class="hljs-keyword">async</span>{ 
  deviceInfo = <span class="hljs-keyword">await</span> IsolateHandle.getDeviceInfo(); 
  SchedulerBinding.instance.addTimingsCallback( 
      Report.onReportTimings 
  ); 
}
</code></pre>
<p data-nodeid="42332">然后在 onReportTimings 中将每一帧数据分别保存到 frames 和 routerFrames ，代码如下：</p>
<pre class="lang-dart" data-nodeid="42333"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">数据处理 </span></span>
<span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> onReportTimings(<span class="hljs-built_in">List</span>&lt;FrameTiming&gt; timings) { 
  <span class="hljs-keyword">for</span> (FrameTiming timing <span class="hljs-keyword">in</span> timings) { 
    frames.addFirst(timing); 
    routerFrames.addFirst(timing); 
  } 
}
</code></pre>
<p data-nodeid="42334">routerFrames 为当前页面路由的帧耗时的队列，frames 为所有帧耗时的队列。有了绘制的每一帧数据后，我们再将数据传递到其他线程进行计算，这里会传递到 IsolateHandle 的 calculateFps 方法，我们具体看下这个方法的计算逻辑。</p>
<pre class="lang-dart" data-nodeid="42335"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">计算当个页面的 fps </span></span>
<span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> isolateCalculateFps( 
    ListQueue&lt;FrameTiming&gt; calculateList, 
    <span class="hljs-built_in">String</span> routerName, 
    <span class="hljs-built_in">String</span> deviceInfo 
    ) { 
  <span class="hljs-keyword">if</span>(calculateList == <span class="hljs-keyword">null</span>){ 
    <span class="hljs-keyword">return</span>; 
  } 
  <span class="hljs-built_in">String</span> fpsStr = <span class="hljs-number">60.</span>toStringAsFixed(<span class="hljs-number">3</span>); 
  <span class="hljs-built_in">int</span> lostNum = <span class="hljs-number">0</span>; 
  <span class="hljs-comment">// flutter 标准渲染频率 </span>
  <span class="hljs-built_in">double</span> standardFps = <span class="hljs-number">1000</span>/<span class="hljs-number">60</span>; 
  <span class="hljs-comment">// 计算多少出现掉帧情况，请注意如果是 33秒，其掉帧为2，用34/16。67下取整。 </span>
  calculateList.forEach((frame) { 
    <span class="hljs-keyword">if</span>(frame.totalSpan.inMilliseconds &gt; standardFps) { <span class="hljs-comment">// 超出 16ms 的帧 </span>
      lostNum = lostNum + ( 
          frame.totalSpan.inMilliseconds/standardFps 
      ).floor(); 
    } 
  }); 
  <span class="hljs-keyword">if</span>(calculateList.length + lostNum &gt; <span class="hljs-number">0</span>) { <span class="hljs-comment">// 尽量避免分母为0情况 </span>
    <span class="hljs-built_in">double</span> fps = ( <span class="hljs-number">60</span> * calculateList.length ) / 
        ( calculateList.length + lostNum ); 
    fpsStr = fps.toStringAsFixed(<span class="hljs-number">3</span>); 
  } 
  <span class="hljs-built_in">print</span>(<span class="hljs-string">'<span class="hljs-subst">${deviceInfo}</span>\t<span class="hljs-subst">${fpsStr}</span>'</span>); 
}
</code></pre>
<p data-nodeid="42336">上述代码中，首先获取标准的一帧绘制时间 16.67 ms（目前这部分是hardcode 60 HZ，后续需要匹配 120 HZ），然后分别计算每一帧的渲染耗时情况，并与 16.67 ms 进行对比，得到掉帧数量。计算掉帧的方式是，用耗时时间除以 16.67 ms 下取整就代表掉帧数量。比如耗时 34 ms，代表掉帧了 2 帧，因为 34 / 16.67 = 2.039。最后用以下公式来计算 FPS 。</p>
<pre class="lang-dart" data-nodeid="42337"><code data-language="dart">( list.length * <span class="hljs-number">60</span> ) / ( list.length + lostNum )
</code></pre>
<p data-nodeid="42338">FPS 和 PV 一样将数据上报到服务端，后续在服务端进行分析。</p>
<p data-nodeid="42339">以上就完成了所有的性能上报功能，接下来我们在某个页面进行尝试，这里选择之前侧边栏的“单图片信息”。</p>
<h4 data-nodeid="42340">应用</h4>
<p data-nodeid="42341">在该类中的 initState 中上报 PV ，并在页面开始加载前，将帧放入到具体的 routerFrames 中，代码如下：</p>
<pre class="lang-dart" data-nodeid="42342"><code data-language="dart"><span class="hljs-meta">@override</span> 
<span class="hljs-keyword">void</span> initState() { 
  <span class="hljs-keyword">super</span>.initState(); 
  <span class="hljs-comment">/// <span class="markdown">开始记录fps </span></span>
  Report.startRecord(<span class="hljs-string">'<span class="hljs-subst">${<span class="hljs-keyword">this</span>.runtimeType}</span>'</span>); 
  indexPos = <span class="hljs-number">0</span>; 
  <span class="hljs-comment">// 拉取推荐内容 </span>
  ApiContentIndex.getRecommendList().then((retInfo) { 
    <span class="hljs-keyword">if</span> (retInfo.ret != <span class="hljs-number">0</span>) { 
      <span class="hljs-comment">// 判断返回是否正确 </span>
      <span class="hljs-keyword">return</span>; 
    } 
    setState(() { 
      contentList = retInfo.data; 
    }); 
  }); 
}
</code></pre>
<p data-nodeid="42343">其中的第 5 行就是上报 PV ，并开始记录 routerFrames ，这里通过 this.runtimeType 可以获得具体的类名。FPS 则在页面最后一帧加载完成后回调，然后在回调中计算 FPS 相关数据。在 Flutter 提供了接收帧绘制完成后回调的方法，需要在 build 逻辑中增加下面的代码。</p>
<pre class="lang-dart" data-nodeid="42344"><code data-language="dart">WidgetsBinding.instance.addPostFrameCallback( 
        (_) =&gt; Report.endRecord(<span class="hljs-string">'<span class="hljs-subst">${<span class="hljs-keyword">this</span>.runtimeType}</span>'</span>) 
);
</code></pre>
<p data-nodeid="42345">然后在 Report.endRecord 调用其他线程函数，计算 FPS，并需要清空 routerFrames 。</p>
<pre class="lang-dart" data-nodeid="42346"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">结束并显示数据 </span></span>
<span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> endRecord(<span class="hljs-built_in">String</span> routerName) { 
  IsolateHandle.calculateFps(routerFrames, routerName, deviceInfo); 
  routerFrames.clear(); 
}
</code></pre>
<p data-nodeid="42347">完成后就可以在虚拟机或者真机上进行模拟测试了，不过这里的 FPS 数据不一定完全准确，后续会进一步再优化更新到<a href="https://github.com/love-flutter/flutter-column" data-nodeid="42510">Github 源码</a>中。</p>
<h3 data-nodeid="42348">总结</h3>
<p data-nodeid="42349">本课时着重介绍了 devTools 中的 Timeline 工具的使用，并且应用该工具带领大家实践分析性能问题，其次也简单介绍了 GPU 的问题，不过 GPU 问题需要根据实际情况来分析。最后通过多线程的方式来计算和上报页面相关的性能数据。</p>
<p data-nodeid="42350">学完本课时，你需要掌握 Timeline 分析 UI 性能问题的方法，并且了解如何捕获 Flutter 页面的绘制耗时数据，以及如何简单计算 FPS。</p>
<p data-nodeid="42351" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="42517">点击链接查看本课时代码</a></p>

---

### 精选评论

##### *鑫：
> 我用devtool分析了自己写的一个页面，看着timeline的图也还是不会找到问题的原因和解决办法，我用顶层组件包了一个provider.下面有一个pageview，我切换页面，pageview消耗时间好长，不知道怎么优化？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 首先你需要注意是在profile运行模式下。如果是在debug模式下会出现性能影响很大的情况。如果在profile还出现这个问题，你需要按照文章说的步骤，你可以照着步骤去走。最重要的是把组件的耗时分析打开，那样你就可以跟踪到具体哪个组件构建比较耗时。

##### **2201：
> 这段代码的例子github上没有找到，如果采用Isolate 线程去进行埋点统计用户操作行为，是否可行？会不会导致性能问题？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不会的哈，这里我是做过测试，因为计算是非常耗CPU，但是如果把这个给到其他线程处理，是不会影响到主进程的性能的。

