<p data-nodeid="65937" class="">上一课时我们完善了首页推荐功能，本课时将完善个人页面。个人页面涉及红点组件的知识点，因此本课时在完善个人页面的同时，会着重介绍下该功能的实现。</p>
<h3 data-nodeid="65938">实现效果</h3>
<p data-nodeid="65939">我们先来看下本课时要完成的一个界面效果，如图 1 动画所示。</p>
<p data-nodeid="65940"><img src="https://s0.lgstatic.com/i/image/M00/37/AD/Ciqc1F8aewCAIDGrAAocYK2ZqIo350.gif" alt="20200712_160246.gif" data-nodeid="66015"><br>
图 1 本课时目标效果</p>
<p data-nodeid="65941">首先在最底部导航栏增加了消息未读数提示，当有新的未读消息时候，会有红点提示。个人界面展示了个人信息以及个人相关的操作栏（我的好友、我的消息和系统设置）。</p>
<p data-nodeid="65942">接下来我们来看下实现该效果需要做哪些前期准备工作。</p>
<h3 data-nodeid="65943">前期准备</h3>
<p data-nodeid="65944">基于图 1 的效果，我们首先要实现个人页面。个人页面是一个新页面，对于新页面我们按照第 7 课时的知识去设计页面。由于新增了红点功能，首先在 App 启动时 ，需要一个新的接口拉取服务器未读消息数，然后新增 API 来拉取该数据。其次这个数据状态，需要在多个组件中共享，因此要新增 Model 来管理该状态 。</p>
<h4 data-nodeid="65945">组件设计</h4>
<p data-nodeid="65946">根据图 2 的界面效果，我们将页面拆分图 3 组件树。</p>
<p data-nodeid="65947"><img src="https://s0.lgstatic.com/i/image/M00/37/B9/CgqCHl8aeySAO1UTAAIF-lgAvBA674.png" alt="image (9).png" data-nodeid="66026"><br>
图 2 个人页面效果</p>
<p data-nodeid="65948"><img src="https://s0.lgstatic.com/i/image/M00/37/AD/Ciqc1F8aezCAA5btAACFxGz_UhM158.png" alt="image (10).png" data-nodeid="66031"><br>
图 3 个人页面组件树设计</p>
<p data-nodeid="65949">图 3 组件树中， 左侧为最上面的头像和昵称，右侧为功能列表 。由于是一个有限的列表，因此可以使用 ListView 来封装组件。具体代码编写部分和之前所介绍的没有太大区别，详细代码可<a href="https://github.com/love-flutter/flutter-column" data-nodeid="66037">前往 github 参考</a>。</p>
<p data-nodeid="65950">个人页面开发完成后，我们再来看下红点功能所涉及的 API 和 Model 功能部分。由于用户信息和红点未读消息，都需要状态共享，因此需要创建两个 Model 类 。 这两个 Model 类的代码逻辑基本一致，下面只介绍与红点未读消息有关的部分。</p>
<h4 data-nodeid="65951">API</h4>
<p data-nodeid="65952">在 App 启动时就需要拉取未读消息数，因此需要一个接口来获取未读消息内容 。在 api/user_info 目录下创建 message.dart 来管理消息接口 ，实现该 ApiUserInfoMessage 类，代码如下：</p>
<pre class="lang-dart" data-nodeid="65953"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取用户消息相关 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ApiUserInfoMessage</span> </span>{ 
  <span class="hljs-comment">/// <span class="markdown">获取自己的个人信息 </span></span>
  <span class="hljs-keyword">static</span> <span class="hljs-built_in">int</span> getUnReadMessageNum() { 
    <span class="hljs-keyword">return</span> <span class="hljs-number">18</span>; 
  } 
}
</code></pre>
<p data-nodeid="65954">上面就是这个 API 的功能，里面包含一个 getUnReadMessageNum 方法，这里模拟返回一个假数据 18 个未读消息。</p>
<h4 data-nodeid="65955">Model</h4>
<p data-nodeid="65956">由于未读消息会被应用在底部导航栏和个人页面两个组件页面，因此需要使用 Provider 来做状态管理，在 model 下创建 new_message_model.dart ，并实现下面代码：</p>
<pre class="lang-dart" data-nodeid="65957"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>; 
<span class="hljs-comment">/// <span class="markdown">name状态管理模块 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">NewMessageModel</span> <span class="hljs-title">with</span> <span class="hljs-title">ChangeNotifier</span> </span>{ 
  <span class="hljs-comment">/// <span class="markdown">系统未读新消息数 </span></span>
  <span class="hljs-built_in">int</span> newMessageNum; 
  <span class="hljs-comment">/// <span class="markdown">构造函数 </span></span>
  NewMessageModel({<span class="hljs-keyword">this</span>.newMessageNum}); 
  <span class="hljs-comment">/// <span class="markdown">获取未读消息 </span></span>
  <span class="hljs-built_in">int</span> <span class="hljs-keyword">get</span> value =&gt; newMessageNum; 
  <span class="hljs-comment">/// <span class="markdown">设置已经阅读消息 </span></span>
  <span class="hljs-keyword">void</span> readNewMessage() { 
    <span class="hljs-keyword">if</span>(newMessageNum == <span class="hljs-number">0</span>) { 
      <span class="hljs-keyword">return</span>; 
    } 
    newMessageNum = <span class="hljs-number">0</span>; 
    notifyListeners(); 
  } 
}
</code></pre>
<p data-nodeid="66512" class="">第 6 行是声明一个未读消息字段，保存未读消息数量。第 9 行是构造函数，第 12 行是设置一个 get 方法，第 14 行是设置已读状态，也可以在此调用服务端，将服务端未读状态清零，同时将本地的未读消息数清零。在写 Model 代码要特别注意，比如上面的第 16 行，目的就是避免没必要的 rebuild，当已经没有未读消息，则不需要处理任何行为。</p>


<p data-nodeid="65959">在完成 API 和 Model 部分代码后，接下来修改入口文件 main.dart，在该入口文件中要多增加一个状态管理模块。</p>
<h4 data-nodeid="65960">Main.dart</h4>
<p data-nodeid="65961">由于需求的改变，现在需要多个共享状态类，课时之前只有一个状态 like_num_model，现在需要新增一个 new_message_model 状态，这里就需要使用到 MultiProvider，避免嵌套多层。需要将 main.dart build 方法修改为下面逻辑：</p>
<pre class="lang-dart" data-nodeid="65962"><code data-language="dart"><span class="hljs-meta">@override</span> 
Widget build(BuildContext context) { 
  <span class="hljs-keyword">return</span> _getProviders( 
    context, 
    MaterialApp( 
        title: <span class="hljs-string">'Two You'</span>, <span class="hljs-comment">// APP 名字 </span>
        debugShowCheckedModeBanner: <span class="hljs-keyword">false</span>, 
        theme: ThemeData( 
          primarySwatch: Colors.blue, <span class="hljs-comment">// APP 主题 </span>
        ), 
        routes: Router().registerRouter(), 
        home: Entrance()), 
  ); 
}
</code></pre>
<p data-nodeid="65963">为了保持代码的整洁，新增了一个 _getProviders 方法，然后将状态管理相关的逻辑放入 _getProviders 中，其他组件相关的逻辑还是在 build 中，具体在看下 _getProviders 中的代码逻辑：</p>
<pre class="lang-dart" data-nodeid="65964"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">部分数据需要获取初始值 </span></span>
Widget _getProviders(BuildContext context, Widget child) { 
  StructUserInfo myUserInfo = ApiUserInfoIndex.getSelfUserInfo(); 
  <span class="hljs-keyword">if</span>(myUserInfo == <span class="hljs-keyword">null</span>){ 
    <span class="hljs-keyword">return</span> CommonError(); 
  } 
  <span class="hljs-built_in">int</span> unReadMessageNum = ApiUserInfoMessage.getUnReadMessageNum(); 
  <span class="hljs-keyword">return</span> MultiProvider( 
    providers: [ 
      ChangeNotifierProvider(create: (context) =&gt; LikeNumModel()), 
      ChangeNotifierProvider( 
          create: (context) =&gt; UserInfoModel( 
            myUserInfo: myUserInfo 
          ) 
      ), 
      ChangeNotifierProvider( 
          create: (context) =&gt; NewMessageModel( 
              newMessageNum: unReadMessageNum 
          ) 
      ), 
    ], 
    child: child, 
  ); 
}
</code></pre>
<p data-nodeid="65965">代码第 3 到 7 行是从服务端调用未读消息接口，并获得用户信息和用户未读消息数 。第 9 行使用 MultiProvider 来封装所有需要状态管理的代码，其中每一个状态管理的格式按照下面的方式：</p>
<pre class="lang-dart" data-nodeid="65966"><code data-language="dart">ChangeNotifierProvider(create: (context) =&gt; LikeNumModel()),
</code></pre>
<p data-nodeid="65967">LikeNumModel 为 Model 类，可以为类增加初始赋值，比如上面的 UserInfoModel 和 NewMessageModel。</p>
<p data-nodeid="65968">通过以上方法，我们就将用户信息和未读消息两个状态进行了组件共享，接下来我们看下如何设计红点组件。</p>
<h3 data-nodeid="65969">红点组件</h3>
<p data-nodeid="65970">在 App 中红点和消息提醒是非常常见的应用，因此需要将该功能，设计为一个基础通用组件。在 Flutter 中是提供了一个比较通用的库 <a href="https://pub.dev/packages/badges" data-nodeid="66077">badges</a>。如果你觉得不太适用也可以自己来封装，本课时主要是基于这个组件库实现一个二次封装应用，先来具体看下二次封装的红点组件实现部分。</p>
<h4 data-nodeid="65971">组件实现</h4>
<p data-nodeid="65972">根据自身的业务，我们可以设计为两种， 一个是只显示红点，另一个是显示具体未读消息数的，先看下只显示红点的部分。</p>
<pre class="lang-dart" data-nodeid="65973"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:badges/badges.dart'</span>; 
<span class="hljs-comment">/// <span class="markdown">通用红点逻辑 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CommonRedMessage</span>  </span>{ 
  <span class="hljs-comment">/// <span class="markdown">只展示红点，不展示具体消息数 </span></span>
  <span class="hljs-keyword">static</span> Widget showRedWidget(Widget needRedWidget, <span class="hljs-built_in">int</span> newMessageNum) { 
    <span class="hljs-keyword">if</span>(newMessageNum &lt; <span class="hljs-number">1</span>) { <span class="hljs-comment">// 小于 1 的消息则无须设置 </span>
      <span class="hljs-keyword">return</span> needRedWidget; 
    } 
    <span class="hljs-keyword">return</span> _getBadge(needRedWidget, <span class="hljs-string">''</span>); 
  } 

  <span class="hljs-comment">/// <span class="markdown">获取 badge 组件 </span></span>
  <span class="hljs-keyword">static</span> Widget _getBadge(Widget needRedWidget, <span class="hljs-built_in">String</span> msgTips) { 
    <span class="hljs-keyword">return</span> Badge( 
      alignment: Alignment.bottomLeft, 
      position: BadgePosition.topRight(), 
      toAnimate: <span class="hljs-keyword">false</span>, 
      badgeContent: Text( 
        <span class="hljs-string">'<span class="hljs-subst">$msgTips</span>'</span>, 
        style: TextStyle( 
          color: Colors.white, 
          fontSize: <span class="hljs-number">10.0</span>, 
          letterSpacing: <span class="hljs-number">1</span>, 
          wordSpacing: <span class="hljs-number">2</span>, 
          height: <span class="hljs-number">1</span>, 
        ), 
      ), 
      child: needRedWidget, 
    ); 
  } 
}
</code></pre>
<p data-nodeid="66896" class="">代码第 7 行的方法 showRedWidget 就是只显示红点提醒，其调用的是 _getBadge 方法，该方法主要是应用 Badge 第三方组件，上面代码中的五个参数的作用分别是：</p>

<ul data-nodeid="65975">
<li data-nodeid="65976">
<p data-nodeid="65977">alignment，child 组件的展示方式，这里是底部靠左；</p>
</li>
<li data-nodeid="65978">
<p data-nodeid="65979">position，红点或者未读消息数的位置，这里是右上角；</p>
</li>
<li data-nodeid="65980">
<p data-nodeid="65981">toAnimate，表示动画，这里直接去掉，感觉效果不太好，也没有必要；</p>
</li>
<li data-nodeid="65982">
<p data-nodeid="65983">badgeContent，则是红点的样式内容，需要 Text 组件；</p>
</li>
<li data-nodeid="65984">
<p data-nodeid="65985">child，就是需要展示红点的组件。</p>
</li>
</ul>
<p data-nodeid="65986">未读消息也是使用到 _getBadge 方法，但这里传入的是具体的消息数，而不是一个空字符，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="65987"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">展示消息提醒 </span></span>
<span class="hljs-keyword">static</span> Widget showRedNumWidget(Widget needRedWidget, <span class="hljs-built_in">int</span> newMessageNum) { 
  <span class="hljs-keyword">if</span>(newMessageNum &lt; <span class="hljs-number">1</span>) { <span class="hljs-comment">// 小于1的消息则无须设置 </span>
    <span class="hljs-keyword">return</span> needRedWidget; 
  } 
  <span class="hljs-comment">// 消息数大于99时，则只显示一个红点即可 </span>
  <span class="hljs-built_in">String</span> msgTips = newMessageNum &gt; <span class="hljs-number">99</span> ? <span class="hljs-string">'99+'</span> : <span class="hljs-string">'<span class="hljs-subst">$newMessageNum</span>'</span>; 
  <span class="hljs-keyword">return</span> _getBadge(needRedWidget, msgTips); 
}
</code></pre>
<p data-nodeid="65988">考虑到未读消息数小于 1 不用展示，其次为了避免消息未读数量过大导致 UI 问题，这里在第 7 行代码也加了判断，具体还是根据业务场景来配置。完成基础组件后，我们再来看下该组件的应用部分。</p>
<h4 data-nodeid="65989">组件应用</h4>
<p data-nodeid="65990">组件的应用包含在两部分，一部分是底部导航栏，另外一部分是个人页的“我的消息”按钮。我们先来看下底部导航栏部分，这部分代码在 pages/entrance.dart 中，我们只看修改的部分。</p>
<pre class="lang-dart" data-nodeid="65991"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取页面内容部分 </span></span>
Widget _getScaffold(BuildContext context) { 
  <span class="hljs-keyword">final</span> newMessageModel = Provider.of&lt;NewMessageModel&gt;(context);
</code></pre>
<p data-nodeid="65992">首先需要通过 Provider 获得 NewMessageModel 的操作句柄。</p>
<pre class="lang-dart" data-nodeid="65993"><code data-language="dart">BottomNavigationBarItem( 
  icon: CommonRedMessage.showRedWidget( 
      Icon(Icons.person), 
      newMessageModel.value 
  ), 
  title: Text(<span class="hljs-string">'我'</span>), 
  activeIcon: CommonRedMessage.showRedWidget( 
      Icon(Icons.person_outline), 
      newMessageModel.value 
  ), 
),
</code></pre>
<p data-nodeid="65994">修改底部导航栏的“我”，将其中的 icon 使用红点组件封装，代码在第 2 到 5 行，这里封装在 icon 上界面效果是最好的 ， 这样就在底部导航栏增加了未读消息红点提醒 。</p>
<p data-nodeid="65995">为了演示红点和消息数两种场景，在导航上使用红点来演示效果，个人页面则演示展示具体未读消息数量。再来看下个人页面的代码部分，这部分逻辑在 widgets/user_page/button_list.dart 中。</p>
<pre class="lang-dart" data-nodeid="65996"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">个人页面的功能列表 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserPageButtonList</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{ 
  <span class="hljs-meta">@override</span> 
  Widget build(BuildContext context) { 
    <span class="hljs-keyword">final</span> newMessageModel = Provider.of&lt;NewMessageModel&gt;(context); 
    <span class="hljs-keyword">return</span> ListView( 
      children: &lt;Widget&gt;[ 
        ListTile( 
          leading: Icon(Icons.person_pin), 
          title: Text(<span class="hljs-string">'我的好友'</span>), 
          onTap: () {}, 
        ), 
        ListTile( 
          leading: CommonRedMessage.showRedNumWidget( 
            Icon(Icons.email), 
            newMessageModel.value 
          ), 
          title: Text(<span class="hljs-string">'我的消息'</span>), 
          onTap: () {}, 
        ), 
        ListTile( 
          leading: Icon(Icons.settings), 
          title: Text(<span class="hljs-string">'系统设置'</span>), 
          onTap: () {}, 
        ) 
      ], 
    ); 
  } 
}
</code></pre>
<p data-nodeid="65997">以上代码中的第 18 行到 21 行，就是将 icon 封装在了红点组件内。未读消息展示已经介绍了，那么我们再来看下如何消除消息红点和未读消息。</p>
<h4 data-nodeid="65998">消除红点</h4>
<p data-nodeid="65999">在 new_messsage_model 状态管理类中，有一个 readNewMessage 方法，该方法就是将未读消息设置为 0 ， 然后通知数据监听方，这里我们将点击行为在个人页面的“我的消息”来触发，将 widgets/user_page/button_list.dart 修改为下面的部分：</p>
<pre class="lang-dart" data-nodeid="66000"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:provider/provider.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/model/new_message_model.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/widgets/common/red_message.dart'</span>; 
<span class="hljs-comment">/// <span class="markdown">个人页面的功能列表 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserPageButtonList</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{ 
  <span class="hljs-meta">@override</span> 
  Widget build(BuildContext context) { 
    <span class="hljs-keyword">final</span> newMessageModel = Provider.of&lt;NewMessageModel&gt;(context); 
    <span class="hljs-keyword">return</span> ListView( 
      children: &lt;Widget&gt;[ 
        ListTile( 
          leading: Icon(Icons.person_pin), 
          title: Text(<span class="hljs-string">'我的好友'</span>), 
          onTap: () {}, 
        ), 
        ListTile( 
          leading: CommonRedMessage.showRedNumWidget( 
            Icon(Icons.email), 
            newMessageModel.value 
          ), 
          title: Text(<span class="hljs-string">'我的消息'</span>), 
          onTap: () { 
            newMessageModel.readNewMessage(); 
          }, 
        ), 
        ListTile( 
          leading: Icon(Icons.settings), 
          title: Text(<span class="hljs-string">'系统设置'</span>), 
          onTap: () {}, 
        ) 
      ], 
    ); 
  } 
}
</code></pre>
<p data-nodeid="66001">上面代码中的第 28 行就是点击触发消息消除，接下来我们运行看下效果，如图 4 的动效所示。</p>
<p data-nodeid="66002"><img src="https://s0.lgstatic.com/i/image/M00/37/AE/Ciqc1F8ae4WASR5LAAocYK2ZqIo031.gif" alt="20200712_160246 (1).gif" data-nodeid="66118"><br>
图 4 红点效果图</p>
<p data-nodeid="66003">以上就实现了红点组件的设计，并应用红点组件完善了 Two You Friend 的个人页面功能。</p>
<h3 data-nodeid="66004">总结</h3>
<p data-nodeid="66005">本课时在实现 App 个人页面的过程中，着重介绍了红点组件的设计和应用，同时介绍到了 Provider 多状态管理的方法。学习完本课时后，你要熟练应用红点组件，并且掌握其业务组件设计的方法，其次需要掌握 Provider 的多状态管理方法 。</p>
<p data-nodeid="66006">在本课时之前，所有的 API 接口都是一个假接口数据，下一课时我们将介绍如何进行网络请求，来完善 API 部分功能。谢谢大家。</p>
<p data-nodeid="66007" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="66127">点击此链接查看本课时源码</a></p>

---

### 精选评论


