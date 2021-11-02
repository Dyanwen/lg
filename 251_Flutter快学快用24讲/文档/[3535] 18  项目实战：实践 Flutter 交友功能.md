<p data-nodeid="115274" class="">本课时我会和大家一起来完善 App 的其他功能，其中包括：我的好友、我的消息、系统设置和搜索功能。按照我们之前课时所学的技术点，我们可以通过绘制组件树+布局来实现，在实现过程中也会介绍一些新的知识点，接下来我们就分别看下这几个功能的实现过程。</p>



<h3 data-nodeid="113888">我的好友</h3>
<p data-nodeid="116364">我们首先看下要实现的效果图，如图 1 所示。</p>
<p data-nodeid="128276"><img src="https://s0.lgstatic.com/i/image/M00/3E/A7/Ciqc1F8tC1OAa6iOAABaIAPOiu0051.png" alt="Drawing 0.png" data-nodeid="128280"></p>
<div data-nodeid="133974" class=""><p style="text-align:center">图 1 我的好友效果图</p> </div>







<p data-nodeid="117994">根据图 1 的效果图，我们绘制出组件树+布局，如图 2 所示。</p>
<p data-nodeid="128761"><img src="https://s0.lgstatic.com/i/image/M00/3E/A7/Ciqc1F8tC3SAJma7AACIdJvSfFY733.png" alt="Drawing 2.png" data-nodeid="128765"></p>
<div data-nodeid="134443" class=""><p style="text-align:center">图 2 组件树</p> </div>







<p data-nodeid="113897">图 2 很清晰地分析出了界面所转化的组件树，由于这里都不涉及动态组件，因此将 Text 和 Image 作为一个 card 组件即可。代码实现逻辑和我们之前介绍的推荐页面和关注页面基本一样，接下来我们看下“我的消息”的实现。</p>
<h3 data-nodeid="113898">我的消息</h3>
<p data-nodeid="119060">我们先来看下“我的消息”要实现的界面效果，如图 3 所示。</p>
<p data-nodeid="129244"><img src="https://s0.lgstatic.com/i/image/M00/3E/B2/CgqCHl8tC4CAKYbEAACBTQQFajM318.png" alt="Drawing 4.png" data-nodeid="129248"></p>
<div data-nodeid="133505" class=""><p style="text-align:center">图 3 我的消息界面效果</p> </div>







<p data-nodeid="120110">根据图 3 的效果图，我们绘制出组件树+布局，如图 4 所示。</p>
<p data-nodeid="129725"><img src="https://s0.lgstatic.com/i/image/M00/3E/B2/CgqCHl8tC4iAA2RQAAGgiSh8hy0630.png" alt="Drawing 6.png" data-nodeid="129729"></p>
<div data-nodeid="133036" class=""><p style="text-align:center">图 4 我的消息组件树+布局</p> </div>







<p data-nodeid="113907">图 4 就非常清晰地描述了我们整个 UI 构造：</p>
<ul data-nodeid="113908">
<li data-nodeid="113909">
<p data-nodeid="113910">图 4 中的 Row-Expanded-1 和 Row-Expanded-5 代表的是使用 flex 布局，左右屏幕占比 1 比 5；</p>
</li>
<li data-nodeid="113911">
<p data-nodeid="113912">图 4 中的 first_line 代表的是图 3 右侧的用户昵称和时间一行；</p>
</li>
<li data-nodeid="113913">
<p data-nodeid="113914">图 4 中的 spaceBetween 是 Row 的 mainAxisAlignment 属性，代表的是两端对齐，具体这部分代码如下。</p>
</li>
</ul>
<pre class="lang-dart" data-nodeid="113915"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取右侧的首行 </span></span>
Widget _getFirstLine() { 
  <span class="hljs-keyword">return</span>  Row( 
    mainAxisAlignment: MainAxisAlignment.spaceBetween, 
    children: &lt;Widget&gt;[ 
      Text( 
        userMessage.userInfo.nickName, 
        style: TextStyles.commonStyle(<span class="hljs-number">0.8</span>, Colors.black), 
      ), 
      _getMessageTimeSection(userMessage.messageTime), 
    ], 
  ); 
} 
</code></pre>
<p data-nodeid="113916">由于这里也没有涉及组件的复用和动态组件，因此这里也建议将整个组件内容设计为一个组件叫作 message_card。为了代码维护性，可以使用类函数来封装小组件，为后续重构抽象为通用组件做准备，例如这里我们将 first_line 设计为一个类函数，如上代码中的 _getFirstLine 函数。</p>
<h3 data-nodeid="113917">系统设置</h3>
<p data-nodeid="121144">接下来我们来看下“系统设置”这部分界面效果，如图 5 所示。</p>
<p data-nodeid="130204"><img src="https://s0.lgstatic.com/i/image/M00/3E/A7/Ciqc1F8tC5mAb80WAABCUd3CwUc272.png" alt="Drawing 8.png" data-nodeid="130208"></p>
<div data-nodeid="132567" class=""><p style="text-align:center">图 5 系统设置的效果</p> </div>







<p data-nodeid="113922">看到图 5 的效果后，其实组件设计可能不是关键。这里涉及两个新的知识点：</p>
<ul data-nodeid="113923">
<li data-nodeid="113924">
<p data-nodeid="113925">在 Flutter 上怎么处理表单数据；</p>
</li>
<li data-nodeid="113926">
<p data-nodeid="113927">怎么保存系统设置的数据。</p>
</li>
</ul>
<p data-nodeid="113928">这里具体的组件树+布局就不绘制了，我们可以将实现过程分为四部分：第三方库引入、通用文件存储、model 应用和组件应用。</p>
<h4 data-nodeid="113929">第三方库</h4>
<p data-nodeid="122732" class="">这里我们需要使用到 Flutter 本地存储功能，Flutter 本地存储功能包含三种：<a href="https://pub.dev/packages/shared_preferences" data-nodeid="122738">shared_preferences</a>、<a href="https://pub.dev/packages/path_provider" data-nodeid="122744">path_provider</a> 文件存储以及 <a href="https://pub.dev/packages/sqflite" data-nodeid="122748">sqflite</a>。这里只介绍 path_provider 文件存储的实现，其他两个大家参照官网的介绍尝试即可。使用该 path_provider  库需要在 pubspec.yaml 中增加库引入，然后更新本地库。</p>



<h4 data-nodeid="113931">通用文件存储</h4>
<p data-nodeid="123266" class="">接下来我们基于 path_provider 实现一个通用的文件内容存储，<a href="https://github.com/love-flutter/flutter-column" data-nodeid="123272">代码在 github 源码中</a>的 util/tools/local_storage.dart 中。这里我们主要需要实现两个方法，一个是文件储存内容，一个文件读取内容。</p>

<ul data-nodeid="113933">
<li data-nodeid="113934">
<p data-nodeid="113935">文件存储</p>
</li>
</ul>
<p data-nodeid="113936">我们先来看下文件存储的逻辑，如下代码：</p>
<pre class="lang-dart" data-nodeid="113937"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">将数据保存到文件中 </span></span>
<span class="hljs-keyword">static</span> Future&lt;<span class="hljs-keyword">void</span>&gt; save(<span class="hljs-built_in">String</span> content, <span class="hljs-built_in">String</span> filePath) <span class="hljs-keyword">async</span> { 
  <span class="hljs-keyword">final</span> directory = <span class="hljs-keyword">await</span> getApplicationDocumentsDirectory(); 
  File file = <span class="hljs-keyword">new</span> File(<span class="hljs-string">'<span class="hljs-subst">${directory.path}</span>/<span class="hljs-subst">$filePath</span>'</span>); 
  file.writeAsString(content); 
} 
</code></pre>
<p data-nodeid="113938">因为是异步获取文件存储路径，因此 save 方法也需要作为异步逻辑，由于无须等待处理结果，因此返回 void。上面代码中使用了 path_provider 的 getApplicationDocumentsDirectory 的方法获取文件存储目录，使用 dart:io 获取具体文件的操作句柄，最后将内容写入文件，接下来我们看下读取的过程。</p>
<ul data-nodeid="113939">
<li data-nodeid="113940">
<p data-nodeid="113941">文件读取</p>
</li>
</ul>
<p data-nodeid="113942">读取的过程和写的代码相似，首先是找到文件并获取文件操作句柄，然后再使用文件句柄读取文件具体内容，代码如下：</p>
<pre class="lang-dart" data-nodeid="113943"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取文件数据内容 </span></span>
<span class="hljs-keyword">static</span> Future&lt;<span class="hljs-built_in">String</span>&gt; <span class="hljs-keyword">get</span>(<span class="hljs-built_in">String</span> filePath) <span class="hljs-keyword">async</span> { 
  <span class="hljs-keyword">try</span> { 
    <span class="hljs-keyword">final</span> directory = <span class="hljs-keyword">await</span> getApplicationDocumentsDirectory(); 
    File file = <span class="hljs-keyword">new</span> File(<span class="hljs-string">'<span class="hljs-subst">${directory.path}</span>/<span class="hljs-subst">$filePath</span>'</span>); 
    <span class="hljs-built_in">bool</span> exist = <span class="hljs-keyword">await</span> file.exists(); 
    <span class="hljs-keyword">if</span>(!exist){ <span class="hljs-comment">// 判断是否存在文件 </span>
      <span class="hljs-keyword">return</span> <span class="hljs-string">''</span>; 
    } 
    <span class="hljs-keyword">return</span> file.readAsString(); 
  } <span class="hljs-keyword">catch</span>(e) { 
    <span class="hljs-keyword">return</span> <span class="hljs-string">''</span>; 
  } 
} 
</code></pre>
<p data-nodeid="113944">上面代码增加了一个异常处理，避免读取失败返回错误数据，因此如果这里判断异常，则返回空字符串。在 catch 逻辑中是需要增加上报来监控告警的，后续我们再来介绍这部分内容。</p>
<h4 data-nodeid="113945">model 应用</h4>
<p data-nodeid="113946">因为系统配置是一个全局状态，需要在多个页面使用，所以我们需要将系统数据保存在 model 中，因此我们在 model 创建</p>
<p data-nodeid="113947">system_config_model.dart 文件。在实现逻辑中，需要先调用 LocalStorage 来获取初始配置，代码如下：</p>
<pre class="lang-dart" data-nodeid="113948"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">构造函数 </span></span>
SystemConfigModel.init(){ 
  LocalStorage.<span class="hljs-keyword">get</span>(<span class="hljs-string">'tyfapp.system.config'</span>).then((configStr){ 
    <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; configInfo = {}; 
    <span class="hljs-keyword">if</span>(configStr == <span class="hljs-keyword">null</span> || configStr == <span class="hljs-string">''</span>) { <span class="hljs-comment">// 判断合法性 </span>
      configInfo = { 
        <span class="hljs-string">"accessMessage"</span> : <span class="hljs-keyword">true</span>, 
        <span class="hljs-string">"tipsDetail"</span> : <span class="hljs-keyword">true</span>, 
        <span class="hljs-string">"soundReminder"</span> : <span class="hljs-keyword">true</span>, 
        <span class="hljs-string">"vibrationReminder"</span> : <span class="hljs-keyword">true</span> 
      }; 
    } <span class="hljs-keyword">else</span> { 
      <span class="hljs-keyword">try</span> { <span class="hljs-comment">// 尝试 json 解析，解析失败直接返回 </span>
        configInfo = 
        json.decode(configStr) <span class="hljs-keyword">as</span> <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt;; 
      } <span class="hljs-keyword">catch</span>(e){ 
        <span class="hljs-keyword">return</span>; 
      } 
    } 
    systemConfig = StructSystemConfig.fromJson(configInfo); 
  }); 
} 
</code></pre>
<p data-nodeid="113949">上面代码 init 为构造函数，其中第 3 行是异步读取文件，获取到文件后存储在共享状态变量 systemConfig 中。为了异常考虑，如果没有获取到文件内容，则将共享状态变量默认设置打开状态，也就是 true 值。有了初始化部分，再修改 main.dart 增加一个新的状态共享，部分如下代码：</p>
<pre class="lang-dart" data-nodeid="113950"><code data-language="dart"><span class="hljs-comment">// 初始化共享状态对象 </span>
LikeNumModel likeNumModel = LikeNumModel(); 
NewMessageModel newMessageNum = NewMessageModel(newMessageNum: <span class="hljs-number">0</span>); 
<span class="hljs-comment">// 异步数据处理 </span>
ApiUserInfoMessage.getUnReadMessageNum(newMessageNum); 
<span class="hljs-comment">// 异步获取系统配置 </span>
SystemConfigModel systemConfigModel = SystemConfigModel.init(); 
<span class="hljs-keyword">return</span> MultiProvider( 
  providers: [ 
    ChangeNotifierProvider(create: (context) =&gt; likeNumModel), 
    ChangeNotifierProvider( 
        create: (context) =&gt; UserInfoModel(myUserInfo: myUserInfo)), 
    ChangeNotifierProvider(create: (context) =&gt; newMessageNum), 
    ChangeNotifierProvider(create: (context) =&gt; systemConfigModel), 
  ], 
  child: child, 
); 
</code></pre>
<p data-nodeid="113951">上面代码的第 7 行就是增加了系统变量的初始化，第 15 行就是增加到状态共享中。接下来我们完善下 system_config_model.dart 代码，为其增加 get 和 save 方法，代码如下：</p>
<pre class="lang-dart" data-nodeid="113952"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">转化为StructSystemConfig结构 </span></span>
StructSystemConfig <span class="hljs-keyword">get</span>() { 
  <span class="hljs-keyword">return</span> systemConfig; 
} 
<span class="hljs-comment">/// <span class="markdown">转化为StructSystemConfig结构 </span></span>
<span class="hljs-built_in">bool</span> getSwitchItem(<span class="hljs-built_in">String</span> switchItem) { 
  <span class="hljs-keyword">if</span>(systemConfig == <span class="hljs-keyword">null</span>) { 
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>; 
  } 
  <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>,<span class="hljs-built_in">dynamic</span>&gt; systemConfigJson = 
  StructSystemConfig.toJson(systemConfig); 
  <span class="hljs-keyword">try</span>{ 
    <span class="hljs-keyword">return</span> systemConfigJson[switchItem] <span class="hljs-keyword">as</span> <span class="hljs-built_in">bool</span>; 
  }<span class="hljs-keyword">catch</span>(e){ 
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>; 
  } 
} 
</code></pre>
<p data-nodeid="113953">代码的第 2 到第 18 行中的两个方法 get 和 getSwitchItem ，其作用都是获取系统配置，前者是获取所有配置，后者是获取具体的某个配置。我们继续看下配置保存的两个方法，代码如下。</p>
<pre class="lang-dart" data-nodeid="113954"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">保存单个数据 </span></span>
<span class="hljs-keyword">void</span> saveOne(<span class="hljs-built_in">String</span> key, <span class="hljs-built_in">bool</span> value) { 
  <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>,<span class="hljs-built_in">dynamic</span>&gt; systemConfigJson = 
    StructSystemConfig.toJson(systemConfig); 
  <span class="hljs-keyword">if</span>(systemConfigJson[key] == value) { 
    <span class="hljs-keyword">return</span>; 
  } 
  systemConfigJson[key] = value; 
  systemConfig = StructSystemConfig.fromJson(systemConfigJson); 
  <span class="hljs-built_in">print</span>(systemConfigJson); 
  LocalStorage.save(json.encode(systemConfigJson), <span class="hljs-string">'tyfapp.system.config'</span>); 
  notifyListeners(); 
} 

<span class="hljs-comment">/// <span class="markdown">整体数据保存 </span></span>
<span class="hljs-keyword">void</span> save(StructSystemConfig newSystemConfig) { 
  <span class="hljs-keyword">if</span>( 
  systemConfig.accessMessage == newSystemConfig.accessMessage &amp;&amp; 
      systemConfig.tipsDetail == newSystemConfig.tipsDetail &amp;&amp; 
      systemConfig.soundReminder == newSystemConfig.soundReminder &amp;&amp; 
      systemConfig.vibrationReminder == newSystemConfig.vibrationReminder 
  ) { 
    <span class="hljs-keyword">return</span>; 
  } 
  systemConfig = newSystemConfig; 
  LocalStorage.save( 
      json.encode(StructSystemConfig.toJson(systemConfig)), 
      <span class="hljs-string">'tyfapp.system.config'</span> 
  ); 
  notifyListeners( 
</code></pre>
<p data-nodeid="113955">代码 save 和 saveOne，分别对应保存整个系统配置数据和保存单个系统配置数据。在两者实现逻辑中，首先都做了前期数据校验判断，避免不必要的 build 操作。在 save 代码逻辑中，需要将数据存储到本地，通过调用 LocaStorage.save 来实现。</p>
<h4 data-nodeid="113956">组件应用</h4>
<p data-nodeid="113957">组件应用部分较为简单，我们先看下 pages/system_page/index.dart 的逻辑，如下：</p>
<pre class="lang-dart" data-nodeid="113958"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:provider/provider.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/model/system_config_model.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/widgets/system_page/switch_card.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/system_config.dart'</span>; 
<span class="hljs-comment">/// <span class="markdown">首页 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SystemConfigPageIndex</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{ 
  <span class="hljs-comment">/// <span class="markdown">构造函数 </span></span>
  <span class="hljs-keyword">const</span> SystemConfigPageIndex(); 
  <span class="hljs-meta">@override</span> 
  Widget build(BuildContext context) { 
    <span class="hljs-keyword">final</span> systemConfigModel = Provider.of&lt;SystemConfigModel&gt;(context); 
    StructSystemConfig systemConfig = systemConfigModel.<span class="hljs-keyword">get</span>(); 
    <span class="hljs-keyword">return</span> Container( 
      padding: EdgeInsets.all(<span class="hljs-number">8</span>), 
      child: Column( 
        children: &lt;Widget&gt;[ 
          SystemPageSwitchCard(itemDesc: <span class="hljs-string">'新消息提醒'</span>, switchItem: <span class="hljs-string">'accessMessage'</span>), 
          SystemPageSwitchCard(itemDesc: <span class="hljs-string">'通知显示详情'</span>, switchItem: <span class="hljs-string">'tipsDetail'</span>), 
          SystemPageSwitchCard(itemDesc: <span class="hljs-string">'声音'</span>, switchItem: <span class="hljs-string">'soundReminder'</span>), 
          SystemPageSwitchCard(itemDesc: <span class="hljs-string">'振动'</span>, switchItem: <span class="hljs-string">'vibrationReminder'</span>) 
         ], 
      ), 
    ); 
  } 
} 
</code></pre>
<p data-nodeid="113959">主要逻辑在 build 中，build 中使用了 widgets/system_page/switch_card.dart ，我们看下这个子组件的实现，代码如下：</p>
<pre class="lang-dart" data-nodeid="113960"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:provider/provider.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/model/system_config_model.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/styles/text_syles.dart'</span>; 
<span class="hljs-comment">/// <span class="markdown">单个系统配置 </span></span>
<span class="hljs-comment">/// 
<span class="markdown">/// [title]为帖子详情内容 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SystemPageSwitchCard</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{ 
  <span class="hljs-comment">/// <span class="markdown">传入的帖子标题 </span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> switchItem; 
  <span class="hljs-comment">/// <span class="markdown">消息提醒文字 </span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> itemDesc; 
  <span class="hljs-comment">/// <span class="markdown">构造函数 </span></span>
  <span class="hljs-keyword">const</span> SystemPageSwitchCard( 
      {Key key, <span class="hljs-keyword">this</span>.itemDesc, <span class="hljs-keyword">this</span>.switchItem} 
      ) : <span class="hljs-keyword">super</span>(key: key); 
  <span class="hljs-meta">@override</span> 
  Widget build(BuildContext context) { 
    <span class="hljs-comment">// 获取操作句柄 </span>
    <span class="hljs-keyword">final</span> systemConfigModel = Provider.of&lt;SystemConfigModel&gt;(context); 
    <span class="hljs-keyword">return</span> Row( 
      mainAxisAlignment: MainAxisAlignment.spaceBetween, 
      children: &lt;Widget&gt;[ 
        Text( 
          itemDesc, 
          style: TextStyles.commonStyle(<span class="hljs-number">1</span>, Colors.black), 
        ), 
        Switch( <span class="hljs-comment">// 选择 </span>
            value: systemConfigModel.getSwitchItem(switchItem), 
            activeTrackColor: Colors.lightBlueAccent, 
            materialTapTargetSize: MaterialTapTargetSize.shrinkWrap, 
            onChanged: (newValue) { <span class="hljs-comment">// 触发状态变化 </span>
              systemConfigModel.saveOne( 
                  switchItem, 
                  !systemConfigModel.getSwitchItem(switchItem) 
              ); 
            } 
        ), 
      ], 
    ); 
  } 
} 
</code></pre>
<p data-nodeid="123788">代码第 34 行使用了 Switch 这个组件，该组件的 value 是通过 systemConfigModel 状态共享类获取，在点击切换时触发状态改变，并调用 systemConfigModel 中的 saveOne 触发依赖组件状态变化。</p>
<p data-nodeid="123789">以上就实现了系统设置的功能，相对其他组件的实现，这部分逻辑较为复杂，涉及了 Flutter 的本地存储 以及 Provider 的应用技术点。</p>

<h3 data-nodeid="113962">搜索</h3>
<p data-nodeid="124796">最后我们来看下搜索功能，前面我们已经实现了一个基本的搜索功能，但是其中的接口部分没有补齐，我们先来看下实际的效果，如图 6 所示。</p>
<p data-nodeid="130681"><img src="https://s0.lgstatic.com/i/image/M00/3E/B2/CgqCHl8tC9yAB1kXAAC_xNdvVn4559.png" alt="Drawing 10.png" data-nodeid="130685"></p>
<div data-nodeid="132098" class=""><p style="text-align:center">图 6 搜索提示和搜索结果效果</p> </div>







<h4 data-nodeid="113967">组件树+布局</h4>
<p data-nodeid="125786">根据图 6 的页面效果，我们来绘制组件树+布局，搜索提示就是一个列表，这里就不绘制了，搜索结果稍微复杂一些，主要看下这部分，绘制结果如图 7 所示。</p>
<p data-nodeid="131156"><img src="https://s0.lgstatic.com/i/image/M00/3E/B3/CgqCHl8tC-aAConRAADrcMIRoXM555.png" alt="Drawing 13.png" data-nodeid="131160"></p>
<div data-nodeid="131629" class=""><p style="text-align:center">图 7 搜索结果页面组件树+布局设计</p> </div>







<p data-nodeid="113972">这个组件树的设计，包含了我们布局设计思想中的 8 个过程，竖横、高宽、上下和左右，具体细节就不再赘述。接下来我们看下这两部分逻辑的实现：搜索提示和搜索结果。</p>
<h4 data-nodeid="113973">搜索提示</h4>
<p data-nodeid="113974">搜索提示较为简单，主要逻辑是从服务端拉取搜索提示接口，并返回一个 ListView 列表结果。具体代码如下：</p>
<pre class="lang-dart" data-nodeid="113975"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取 suggest list组件列表 </span></span>
Future&lt;Widget&gt; _getSuggestList() <span class="hljs-keyword">async</span>{ 
  <span class="hljs-built_in">List</span>&lt;<span class="hljs-built_in">String</span>&gt; suggests = <span class="hljs-keyword">await</span> ApiSearchIndex.suggest(query); 
  <span class="hljs-keyword">if</span>(suggests == <span class="hljs-keyword">null</span> || suggests.length &lt; <span class="hljs-number">1</span>){ <span class="hljs-comment">// 异常处理 </span>
    <span class="hljs-keyword">return</span> Container(); 
  } 
  <span class="hljs-comment">// 保留前 5 个搜索 </span>
  <span class="hljs-built_in">int</span> subLen = suggests.length &gt; <span class="hljs-number">5</span> ? <span class="hljs-number">5</span> : suggests.length; 
  <span class="hljs-built_in">List</span>&lt;<span class="hljs-built_in">String</span>&gt; subSuggests = suggests.sublist(<span class="hljs-number">0</span>, subLen); 
  <span class="hljs-keyword">return</span> ListView.builder( 
      scrollDirection: Axis.vertical, 
      shrinkWrap: <span class="hljs-keyword">true</span>, 
      itemCount: subSuggests.length, 
      itemBuilder: (context,index){ 
        <span class="hljs-keyword">return</span>  ListTile( 
            title: RichText( 
                text: TextSpan( 
                  <span class="hljs-comment">// 获取搜索框内输入的字符串，设置它的颜色并加粗 </span>
                    text: subSuggests[index], 
                    style: TextStyles.commonStyle() 
                ) 
            ), 
            onTap: () { 
              query = subSuggests[index]; 
              showResults(context); 
            }, 
        ); 
      } 
  ); 
} 
</code></pre>
<p data-nodeid="113976">代码中，首先使用 query 关键词获取用户输入，通过 ApiSearchIndex.suggest 方法获取服务端搜索提示结果，接下来做一些数据校验，最后根据搜索提示 build 出相应的组件。其中的第 26 行至第 28 行代码的作用是，通过点击搜索提示触发搜索行为，第 27 行替换搜索提示内容，第 28 行执行搜索并获取搜索结果。</p>
<h4 data-nodeid="113977">搜索结果</h4>
<p data-nodeid="113978">根据图 7 的绘制结果，我们了解到这里需要设计两个组件，组件一是展示搜索到的用户列表内容，组件二是展示搜索到的帖子列表内容。我们这里就使用两个组件函数来实现，主要看下用户部分（帖子部分逻辑相似）。</p>
<pre class="lang-dart" data-nodeid="113979"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取用户搜索结果组件 </span></span>
Widget _getUserListWidget(<span class="hljs-built_in">List</span>&lt;StructUserInfo&gt; userList) { 
  <span class="hljs-keyword">if</span>(userList == <span class="hljs-keyword">null</span> || userList.length &lt; <span class="hljs-number">1</span>){ 
    <span class="hljs-keyword">return</span> Container(); 
  } 
  <span class="hljs-built_in">int</span> subLen = userList.length &gt; <span class="hljs-number">5</span> ? <span class="hljs-number">5</span> : userList.length; 
  <span class="hljs-built_in">List</span>&lt;StructUserInfo&gt; subUserList = userList.sublist(<span class="hljs-number">0</span>, subLen); 
  <span class="hljs-keyword">return</span> ListView.builder( 
      scrollDirection: Axis.vertical, 
      shrinkWrap: <span class="hljs-keyword">true</span>, 
      itemCount: subUserList.length + <span class="hljs-number">1</span>, 
      itemBuilder: (context,index) { 
        <span class="hljs-keyword">if</span>(index == <span class="hljs-number">0</span>){ 
          <span class="hljs-keyword">return</span> Row( 
            children: &lt;Widget&gt;[ 
              Padding(padding: EdgeInsets.only(left: <span class="hljs-number">10</span>)), 
              Text( 
                <span class="hljs-string">'用户'</span>, 
                style: TextStyles.commonStyle(<span class="hljs-number">0.9</span>), 
              ), 
            ], 
          ); 
        } 
        <span class="hljs-keyword">return</span> UserPageCard(userInfo: subUserList[index - <span class="hljs-number">1</span>]); 
      }); 
} 
</code></pre>
<p data-nodeid="126284">以上组件代码的实现与我们之前所学习的知识点，没有太大的差异性。核心知识点是应用 ListView.builder 组件，来显示 seaction_name （也就是上面的 Text 组件）和搜索结果中的用户列表（上面的 UserPageCard 组件）。</p>
<p data-nodeid="127785" class="">以上就完成了搜索部分的逻辑，具体代码<a href="https://github.com/love-flutter/flutter-column" data-nodeid="127793">查看 github 中的 pages/search_page/custom_delegate.dart 文件。</a></p>




<h3 data-nodeid="113981">总结</h3>
<p data-nodeid="113982">本课时带领大家实践开发了四个核心页面（我的好友、我的消息、系统设置和搜索）。学完本课时你需要进一步掌握组件树+布局的设计思想，同时掌握 Flutter 本地存储的技术点，进一步巩固 Flutter 编码风格。学完本课时之后，我建议你自行去实现“我的消息”中的私信功能和评论相关的部分（后续会在 github 上提供源码）。</p>
<p data-nodeid="113983">本课时之前，我们对 App 的安全并没有关注太多，可以说完全放任。下一课时我们将通过工具化的方式来上报异常，保证我们 App 的质量，提前发现并解决问题。</p>
<p data-nodeid="113984"><a href="https://github.com/love-flutter/flutter-column" data-nodeid="114163">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### **露：
> 代码数据没有，没发运行，有什么办法解决吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 使用假数据；2. 使用mock接口，https://www.fastmock.site/

