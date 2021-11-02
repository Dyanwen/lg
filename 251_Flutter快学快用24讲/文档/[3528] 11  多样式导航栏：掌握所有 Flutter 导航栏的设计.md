<p data-nodeid="47173" class="">本课时将介绍一些比较通用的导航栏功能，并应用上一课时的知识来实现导航栏跳转对应页面的功能。</p>
<h3 data-nodeid="47174">导航栏样式效果</h3>
<p data-nodeid="47175">目前较为常见三种导航栏功能：底部导航栏、顶部导航栏和侧边导航栏。为了更好的界面效果，我在导航栏基础上增加了搜索功能模块的实现，完善了整个界面交互效果。我们先来看看这三种导航栏+搜索功能的运行效果。</p>
<p data-nodeid="47176"><img src="https://s0.lgstatic.com/i/image/M00/32/E7/CgqCHl8O3yKAXVpIAAr_4vHP8yw482.gif" alt="11.gif" data-nodeid="47241"><br>
图 1 底部导航栏+搜索栏</p>
<p data-nodeid="47177"><img src="https://s0.lgstatic.com/i/image/M00/32/E7/CgqCHl8O3y6ANmY5ABTLVrhmzuE224.gif" alt="22.gif" data-nodeid="47246"><br>
图 2 顶部导航栏+搜索栏</p>
<p data-nodeid="47178"><img src="https://s0.lgstatic.com/i/image/M00/32/DC/Ciqc1F8O3z-ARRi7ABZtQ8SwZzo652.gif" alt="33.gif" data-nodeid="47251"><br>
图 3 侧边栏+搜索栏+底部导航栏</p>
<p data-nodeid="47179"><img src="https://s0.lgstatic.com/i/image/M00/32/E7/CgqCHl8O30yAND7IABhCW3e0Ytk015.gif" alt="44.gif" data-nodeid="47256"><br>
图 4 侧边栏+搜索栏+顶部导航栏</p>
<p data-nodeid="47180">上面就是本课时所需要实现的全部导航栏功能，接下来我们就逐个介绍下如何实现该功能。</p>
<h3 data-nodeid="47181">导航栏实现</h3>
<p data-nodeid="47182">导航栏功能会涉及 Flutter 中几个核心点，我们使用如下表格的方式来说明，后续内容遇到相应的知识点后，可以直接对照表格 1 。</p>
<p data-nodeid="47183"><img src="https://s0.lgstatic.com/i/image/M00/32/AC/Ciqc1F8Os9eAcJO5AAB1aFqjuhE322.png" alt="image (5).png" data-nodeid="47264"></p>
<div data-nodeid="47184"><p style="text-align:center">图 1 底部导航栏+搜索栏</p></div>
<p data-nodeid="47185">可以看到上述导航栏都是 Scaffold 的一个属性，这就类似于一个架子，架子提供了很多模块。如果我们需要某些模块，只需要按照模块的格式插入数据，就可以实现相应功能。这个控件的一些参数应用具体如下。</p>
<pre class="lang-dart" data-nodeid="47186"><code data-language="dart"><span class="hljs-keyword">const</span> Scaffold({
  Key key,
  <span class="hljs-keyword">this</span>.appBar, <span class="hljs-comment">// 应用栏，显示在顶部，包括其中的搜索框</span>
  <span class="hljs-keyword">this</span>.body, <span class="hljs-comment">// 页面的主题显示内容</span>
  <span class="hljs-keyword">this</span>.floatingActionButton, <span class="hljs-comment">// 设置显示在上层区域的按钮，默认位置位于右下角</span>
  <span class="hljs-keyword">this</span>.floatingActionButtonLocation, <span class="hljs-comment">// 设置floatingActionButton的位置</span>
  <span class="hljs-keyword">this</span>.floatingActionButtonAnimator, <span class="hljs-comment">// floatingActionButton动画</span>
  <span class="hljs-keyword">this</span>.persistentFooterButtons, <span class="hljs-comment">// 在底部导航栏之上的一组操作按钮</span>
  <span class="hljs-keyword">this</span>.drawer, <span class="hljs-comment">// 左侧导航栏</span>
  <span class="hljs-keyword">this</span>.endDrawer, <span class="hljs-comment">// 右侧导航栏</span>
  <span class="hljs-keyword">this</span>.bottomNavigationBar, <span class="hljs-comment">// 底部导航栏</span>
  <span class="hljs-keyword">this</span>.bottomSheet, <span class="hljs-comment">// 底部可隐藏导航栏</span>
  <span class="hljs-keyword">this</span>.backgroundColor, <span class="hljs-comment">// 内容区域颜色</span>
  <span class="hljs-keyword">this</span>.resizeToAvoidBottomPadding, <span class="hljs-comment">// 是否重新布局来避免底部被覆盖了，比如当键盘显示的时候，重新布局避免被键盘盖住内容。默认值为 true。</span>
  <span class="hljs-keyword">this</span>.resizeToAvoidBottomInset, <span class="hljs-comment">//键盘弹出时是否重新绘制，以避免输入框被遮挡</span>
  <span class="hljs-keyword">this</span>.primary = <span class="hljs-keyword">true</span>, <span class="hljs-comment">// 是否计算手机顶部状态栏的高度</span>
  <span class="hljs-keyword">this</span>.drawerDragStartBehavior = DragStartBehavior.start, <span class="hljs-comment">// 拖动的处理</span>
  <span class="hljs-keyword">this</span>.extendBody = <span class="hljs-keyword">false</span>, <span class="hljs-comment">// 是否延伸body至底部</span>
  <span class="hljs-keyword">this</span>.extendBodyBehindAppBar = <span class="hljs-keyword">false</span>, <span class="hljs-comment">// 是否延伸body至顶部</span>
  <span class="hljs-keyword">this</span>.drawerScrimColor, <span class="hljs-comment">// 抽屉遮罩层背景色</span>
  <span class="hljs-keyword">this</span>.drawerEdgeDragWidth, <span class="hljs-comment">// 滑动拉出抽屉的生效距离</span>
  <span class="hljs-keyword">this</span>.drawerEnableOpenDragGesture = <span class="hljs-keyword">true</span>, <span class="hljs-comment">// 确定是否可以通过拖动手势打开Scaffold.drawer, 默认情况下，拖动手势处于启用状态</span>
  <span class="hljs-keyword">this</span>.endDrawerEnableOpenDragGesture = <span class="hljs-keyword">true</span>, <span class="hljs-comment">// 确定是否可以使用拖动手势打开Scaffold.endDrawer，默认情况下，拖动手势处于启用状态。</span>
})
</code></pre>
<h4 data-nodeid="47187">1. 底部导航栏</h4>
<p data-nodeid="47188">根据控件 Scaffold 的说明，其中涉及 bottomNavigationBar 这个属性名，在表格 1 中有说明到该属性对应的是一个 BottomNavigationBar 组件，该组件的属性也比较多，如下所示。</p>
<pre class="lang-dart" data-nodeid="47189"><code data-language="dart">BottomNavigationBar({
  Key key,
  <span class="hljs-meta">@required</span> <span class="hljs-keyword">this</span>.items, <span class="hljs-comment">// 数组，对应于BottomNavigationBarItem这个组件为菜单栏的每一项，其中包含四个属性icon、title、activeIcon和backgroundColor</span>
  <span class="hljs-keyword">this</span>.onTap, <span class="hljs-comment">// 点击触发逻辑，一般用来触发页面的跳转更新</span>
  <span class="hljs-keyword">this</span>.currentIndex = <span class="hljs-number">0</span>, <span class="hljs-comment">// 当前所在的 items 数组中的位置</span>
  <span class="hljs-keyword">this</span>.elevation = <span class="hljs-number">8.0</span>, <span class="hljs-comment">// 设置阴影效果值</span>
  BottomNavigationBarType type, <span class="hljs-comment">// fixed(固定位置)和shifting(浮动效果)</span>
  Color fixedColor, <span class="hljs-comment">// 代表选中时候的颜色，不能和selectedItemColor一起使用</span>
  <span class="hljs-keyword">this</span>.backgroundColor, <span class="hljs-comment">// 背景颜色</span>
  <span class="hljs-keyword">this</span>.iconSize = <span class="hljs-number">24.0</span>, <span class="hljs-comment">// icon 大小</span>
  Color selectedItemColor, <span class="hljs-comment">// 代表选中的颜色，不能和selectedItemColor一起使用</span>
  <span class="hljs-keyword">this</span>.unselectedItemColor, <span class="hljs-comment">// 未选中时颜色</span>
  <span class="hljs-keyword">this</span>.selectedIconTheme = <span class="hljs-keyword">const</span> IconThemeData(), <span class="hljs-comment">// 当前选中的BottomNavigationBarItem.icon中图标的大小，不透明度和颜色</span>
  <span class="hljs-keyword">this</span>.unselectedIconTheme = <span class="hljs-keyword">const</span> IconThemeData(), <span class="hljs-comment">// 当前未选中的BottomNavigationBarItem.icon中图标的大小，不透明度和颜色</span>
  <span class="hljs-keyword">this</span>.selectedFontSize = <span class="hljs-number">14.0</span>, <span class="hljs-comment">// 选中的字体大小</span>
  <span class="hljs-keyword">this</span>.unselectedFontSize = <span class="hljs-number">12.0</span>, <span class="hljs-comment">// 未选中字体大小</span>
  <span class="hljs-keyword">this</span>.selectedLabelStyle, <span class="hljs-comment">// 选中字体样式</span>
  <span class="hljs-keyword">this</span>.unselectedLabelStyle, <span class="hljs-comment">// 未选中字体样式</span>
  <span class="hljs-keyword">this</span>.showSelectedLabels = <span class="hljs-keyword">true</span>, <span class="hljs-comment">// 是否开启选中的样式</span>
  <span class="hljs-built_in">bool</span> showUnselectedLabels, <span class="hljs-comment">// 是否开启未选中的样式</span>
})
</code></pre>
<p data-nodeid="47190">介绍完一些基础属性以后，我们来尝试实现顶部导航栏功能。基于上一课时我们实现的两个页面功能，现在我们需要使用导航栏的方式来支持页面跳转。底部导航栏需要一个状态属性 indexValue 来控制导航栏显示位置，我们看下具体在 Scaffold 中的代码逻辑。</p>
<pre class="lang-dart" data-nodeid="47191"><code data-language="dart"><span class="hljs-keyword">return</span> Scaffold(
  appBar: AppBar(
    title: Text(<span class="hljs-string">'Two You'</span>), <span class="hljs-comment">// 页面名字</span>
  ),
  body: Stack(
    children: &lt;Widget&gt;[
      _getPagesWidget(<span class="hljs-number">0</span>),
      _getPagesWidget(<span class="hljs-number">1</span>),
      _getPagesWidget(<span class="hljs-number">2</span>)
    ],
  ),
  bottomNavigationBar: BottomNavigationBar(
    items: [
      BottomNavigationBarItem(
        icon: Icon(Icons.people),
        title: Text(<span class="hljs-string">'推荐'</span>),
        activeIcon: Icon(Icons.people_outline),
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.favorite),
        title: Text(<span class="hljs-string">'关注'</span>),
        activeIcon: Icon(Icons.favorite_border),
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.person),
        title: Text(<span class="hljs-string">'我'</span>),
        activeIcon: Icon(Icons.person_outline),
      ),
    ],
    iconSize: <span class="hljs-number">24</span>,
    currentIndex: _indexNum,
    <span class="hljs-comment">/// <span class="markdown">选中后，底部BottomNavigationBar内容的颜色(选中时，默认为主题色)</span></span>
    <span class="hljs-comment">/// <span class="markdown">（仅当type: BottomNavigationBarType.fixed,时生效）</span></span>
    fixedColor: Colors.lightBlueAccent,
    type: BottomNavigationBarType.fixed,
    onTap: (<span class="hljs-built_in">int</span> index) {
      <span class="hljs-comment">///<span class="markdown">这里根据点击的index来显示，非index的page均隐藏</span></span>
      <span class="hljs-keyword">if</span>(_indexNum != index){
        setState(() {
          _indexNum = index;
        });
      }
    },
  ),
);
</code></pre>
<p data-nodeid="47192">上面代码中，第 5 - 10 行是获取具体的页面信息，并且在 _getPagesWidget 里会判断当前 index 的值，判断当前索引 _indexNum 与 index 是否相同，相同则显示页面，不相同则页面隐藏，具体 _getPagesWidget 代码实现逻辑如下：</p>
<pre class="lang-dart" data-nodeid="47193"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取页面组件</span></span>
Widget _getPagesWidget(<span class="hljs-built_in">int</span> index) {
  <span class="hljs-built_in">List</span>&lt;Widget&gt; widgetList = [
    router.getPageByRouter(<span class="hljs-string">'homepage'</span>),
    Icon(Icons.directions_transit),
    router.getPageByRouter(<span class="hljs-string">'userpage'</span>)
  ];
  <span class="hljs-keyword">return</span> Offstage(
    offstage: _indexNum != index,
    child: TickerMode(
      enabled: _indexNum == index,
      child: widgetList[index],
    ),
  );
}
</code></pre>
<p data-nodeid="47194">上面代码中又使用到了 router 的一个新方法，该方法组件是获取对应 router 名称的组件页面信息，具体代码在 router 中实现，可以参考 github 源码，没有特殊性。</p>
<p data-nodeid="47195">Scaffold 中代码的第 12 行开始实现底部导航栏逻辑，其中使用到了 BottomNavigationBar 控件，配置控件中的 items 属性，该属性注意是导航栏具体每一项数据，iconSize、currentIndex、fixedColor、type 和 onTap，onTap 主要是来切换页面，触发 setState ，然后重新 build 页面结构。</p>
<p data-nodeid="47196">以上就完成了导航栏的设计，运行完以后，就可以正常进行页面切换操作。但是这里存在一些问题，比如在我们上一课时提到的外部拉起 APP 功能，如果拉起的是首页，我们不应该再去 push 一个新的页面，而是打开首页并且根据具体的页面跳转到具体的 tab 下，因此这里需要将 router 中的 push 进行修改。</p>
<p data-nodeid="47197">我们将原来的 push 改为 open，并且对代码做了修改，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="47198"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">执行页面跳转</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 需要特别注意以下逻辑</span></span>
<span class="hljs-comment">/// <span class="markdown">-1 不在首页，则执行跳转</span></span>
<span class="hljs-comment">/// <span class="markdown">大于 -1 则为首页，需要在首页进行 tab 切换，而不是进行跳转</span></span>
<span class="hljs-built_in">int</span> open(BuildContext context, <span class="hljs-built_in">String</span> url) {
  <span class="hljs-comment">// 非entrance入口标识</span>
  <span class="hljs-built_in">int</span> notEntrancePageIndex = <span class="hljs-number">-1</span>;
  <span class="hljs-keyword">if</span> (url.startsWith(<span class="hljs-string">'https://'</span>) || url.startsWith(<span class="hljs-string">'http://'</span>)) {
    <span class="hljs-comment">// 打开网页</span>
    Navigator.push(context, MaterialPageRoute(builder: (context) {
      <span class="hljs-keyword">return</span> CommonWebViewPage(url: url);
    }));
    <span class="hljs-keyword">return</span> notEntrancePageIndex;
  }
  <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; urlParseRet = _parseUrl(url);
  <span class="hljs-built_in">int</span> entranceIndex = routerMapping[urlParseRet[<span class="hljs-string">'action'</span>]].entranceIndex;
  <span class="hljs-keyword">if</span> (entranceIndex &gt; notEntrancePageIndex) {
    <span class="hljs-comment">// 判断为首页，返回切换的tab信息</span>
    <span class="hljs-keyword">return</span> entranceIndex;
  }
  Navigator.pushNamedAndRemoveUntil(context, urlParseRet[<span class="hljs-string">'action'</span>].toString(),
      (route) {
    <span class="hljs-keyword">if</span> (route.settings.name == urlParseRet[<span class="hljs-string">'action'</span>].toString()) {
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
    }
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
  }, arguments: urlParseRet[<span class="hljs-string">'params'</span>]);
  <span class="hljs-comment">// 执行跳转，非首页</span>
  <span class="hljs-keyword">return</span> notEntrancePageIndex;
}
</code></pre>
<p data-nodeid="47199">上面代码与我们之前唯一的不同在于，判断是否在 entrance 页面，如果是则返回相应 tab 的 index，而不是直接进行跳转。如果不是则进行跳转，并返回一个 -1 notEntrancePageIndex。因为返回不一样，因此在 entrance.dart 中也需要对返回的信息做一定的处理，处理部分代码如下。</p>
<pre class="lang-dart" data-nodeid="47200"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">跳转页面</span></span>
<span class="hljs-keyword">void</span> redirect(<span class="hljs-built_in">String</span> link) {
  <span class="hljs-keyword">if</span> (link == <span class="hljs-keyword">null</span>) {
    <span class="hljs-keyword">return</span>;
  }
  <span class="hljs-built_in">int</span> indexNum = router.open(context, link);
  <span class="hljs-keyword">if</span> (indexNum &gt; <span class="hljs-number">-1</span> &amp;&amp; _indexNum != indexNum) {
    setState(() {
      _indexNum = indexNum;
    });
  }
}
</code></pre>
<p data-nodeid="47201">代码主要是判断是否返回非 -1 以及两个 index 不相等，这时候就使用 setState 来切换导航栏 tab。</p>
<h4 data-nodeid="47202">2. 顶部导航栏</h4>
<p data-nodeid="47203">在表格 1 中我们看到顶部导航栏，需要控件 Scaffold 属性 appBar ，在 appBar 中设置 bottom 就可以实现顶部导航栏功能。接下来看下 bottom 的设置方法，代码如下：</p>
<pre class="lang-dart" data-nodeid="47204"><code data-language="dart"><span class="hljs-keyword">return</span> Scaffold(
  appBar: AppBar(
    title: Text(<span class="hljs-string">'Two You'</span>), <span class="hljs-comment">// 页面名字</span>
    bottom: TabBar(
      controller: _controller,
      tabs: &lt;Widget&gt;[
        Tab(
          icon: Icon(Icons.view_list),
          text: <span class="hljs-string">'推荐'</span>,
        ),
        Tab(
          icon: Icon(Icons.favorite),
          text: <span class="hljs-string">'关注'</span>,
        ),
        Tab(
          icon: Icon(Icons.person),
          text: <span class="hljs-string">'我'</span>,
        ),
      ],
    ),
  ),
  body: TabBarView(
    controller: _controller,
    children: [
      router.getPageByRouter(<span class="hljs-string">'homepage'</span>),
      Icon(Icons.directions_transit),
      router.getPageByRouter(<span class="hljs-string">'userpage'</span>)
    ],
  ),
);
</code></pre>
<p data-nodeid="47205">在上面代码中的第 4 到第 21 行是在设置 bottom 的 TabBar 组件。在 TabBar 中，包含了一个控制导航栏的 controller 和具体导航栏的配置信息的 Tabs。在代码第 22 行到第 29 行也是在配置各个 tab 对应的页面内容组件，这里也是通过 controller 来控制显示，具体 controller 控制部分代码如下。</p>
<pre class="lang-dart" data-nodeid="47206"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">跳转页面</span></span>
<span class="hljs-keyword">void</span> redirect(<span class="hljs-built_in">String</span> link) {
  <span class="hljs-keyword">if</span> (link == <span class="hljs-keyword">null</span>) {
    <span class="hljs-keyword">return</span>;
  }
  <span class="hljs-built_in">int</span> indexNum = router.open(context, link);
  <span class="hljs-keyword">if</span> (indexNum &gt; <span class="hljs-number">-1</span> &amp;&amp; _controller.index != indexNum) {
    _controller.animateTo(indexNum);
  }
}
</code></pre>
<p data-nodeid="47207">顶部导航栏的跳转逻辑部分和底部导航栏相似，这里是使用状态变量 _controller 的 animateTo 方法来处理 tab 的切换。其他部分代码改动和底部导航栏都基本一致，具体代码参考 github 源码。</p>
<h4 data-nodeid="47208">3. 侧边导航栏</h4>
<p data-nodeid="47209">侧边栏在表格 1 中，可以看到使用的是 Scaffold 的 drawer 属性。该属性需要一个 Drawer 对象，因此我们在 Widgets 目录中创建一个 menu 目录，并新增 draw.dart 文件，具体代码如下。</p>
<pre class="lang-dart" data-nodeid="47210"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/router.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">左侧菜单</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MenuDraw</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">外部跳转</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">Function</span> redirect;
  <span class="hljs-comment">/// <span class="markdown">默认构造函数</span></span>
  <span class="hljs-keyword">const</span> MenuDraw(<span class="hljs-keyword">this</span>.redirect);
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> Drawer(
        child: MediaQuery.removePadding(
          context: context,
          child: ListView(
            children: &lt;Widget&gt;[
              ListTile(
                leading: Icon(Icons.view_list),
                title: Text(<span class="hljs-string">'推荐'</span>),
                onTap: () {
                  Navigator.pop(context);
                  redirect(<span class="hljs-string">'tyfapp://homepage'</span>);
                },
              ),
              ListTile(
                leading: Icon(Icons.favorite),
                title: Text(<span class="hljs-string">'关注'</span>),
                onTap: () {
                  Navigator.pop(context);
                  Router().open(context, <span class="hljs-string">'http://www.qq.com'</span>);
                },
              ),
              ListTile(
                leading: Icon(Icons.person),
                title: Text(<span class="hljs-string">'我'</span>),
                onTap: () {
                  Navigator.pop(context);
                  redirect(<span class="hljs-string">'tyfapp://userpage'</span>);
                },
              ),
            ],
          ),
        ),
    );
  }
}
</code></pre>
<p data-nodeid="47211">前 4 行是导入相应的库，创建 MenuDraw 类，类包含 redirect 方法，该方法就是 entrance 中声明的 tab 导航栏切换的方法，如果非 entrance 的切换则需要使用到 router 跳转，类似上面代码中的第 33 行 。</p>
<p data-nodeid="47212">代码的第 19 行到第 44 行则为相应的左侧导航栏的配置，onTap 为导航栏的跳转逻辑，在点击相应的 Tap 以后，需要使用 Navigator.pop(context) 来关闭左侧导航栏。</p>
<p data-nodeid="47213">实现完成该 MenuDraw 类后，我们需要在控件 Scaffold 中增加 drawer 属性，代码如下。</p>
<pre class="lang-dart" data-nodeid="47214"><code data-language="dart"><span class="hljs-keyword">return</span> Scaffold(
  appBar: AppBar(
    title: Text(<span class="hljs-string">'Two You'</span>), <span class="hljs-comment">// 页面名字</span>
  ),
  drawer: MenuDraw(redirect),
  ...
);
</code></pre>
<p data-nodeid="47215">上面代码的第 5 行就是新增 drawer 左侧导航栏。</p>
<h4 data-nodeid="47216">4. 搜索功能</h4>
<p data-nodeid="47217">为了让功能更完善，我们需要增加一个右侧搜索功能，这里就涉及表格 1 中 AppBar 的 actions 属性，我们可以在 AppBar 中增加如下代码：</p>
<pre class="lang-dart" data-nodeid="47218"><code data-language="dart">AppBar(
  title: Text(<span class="hljs-string">'Two You'</span>), <span class="hljs-comment">// 页面名字</span>
  actions: &lt;Widget&gt;[
    IconButton(
      icon: Icon(Icons.search),
      onPressed: () {
        showSearch(
            context: context,
            delegate: SearchPageCustomDelegate()
        );
      },
    ),
  ],
)
</code></pre>
<p data-nodeid="47219">在 actions 中可以添加一组功能按钮，由于这里我们只需要搜索功能按钮，因此在 actions 属性中添加一个 IconButton 即可。IconButton 中需要展示一个搜索 icon ，并且点击以后前往搜索页面。</p>
<p data-nodeid="47220">接下来我们就需要实现 SearchPageCustomDelegate 的页面逻辑，新增 search_page 页面，并在 search_page 下新建 custom_delegate.dart 文件，接下来实现该文件代码。</p>
<p data-nodeid="47221">这个类需要继承 SearchDelegate ，然后必须包含四个方法的实现逻辑，代码如下。</p>
<pre class="lang-dart" data-nodeid="47222"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">搜索框</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SearchPageCustomDelegate</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">SearchDelegate</span> </span>{
  <span class="hljs-meta">@override</span>
  <span class="hljs-built_in">List</span>&lt;Widget&gt; buildActions(BuildContext context) {
  }
  <span class="hljs-meta">@override</span>
  Widget buildLeading(BuildContext context) {
  }
  <span class="hljs-meta">@override</span>
  Widget buildResults(BuildContext context) {
  }
  <span class="hljs-meta">@override</span>
  Widget buildSuggestions(BuildContext context) {
  }
}
</code></pre>
<p data-nodeid="47223">buildActions 为右侧的图标按钮，一般我们可以显示一个清除搜索框内容的功能，我们可以使用如下代码来实现。</p>
<pre class="lang-dart" data-nodeid="47224"><code data-language="dart"><span class="hljs-keyword">return</span> [
  IconButton(
    tooltip: <span class="hljs-string">'Clear'</span>,
    icon: <span class="hljs-keyword">const</span> Icon(Icons.clear),
    onPressed: () {
      query = <span class="hljs-string">''</span>;
      showSuggestions(context);
    },
  )
];
</code></pre>
<p data-nodeid="47225">buildLeading 为左侧的按钮一般来触发返回操作，代码实现如下：</p>
<pre class="lang-dart" data-nodeid="47226"><code data-language="dart"><span class="hljs-meta">@override</span>
Widget buildLeading(BuildContext context) {
  <span class="hljs-keyword">return</span> IconButton(
    tooltip: <span class="hljs-string">'Back'</span>,
    icon: AnimatedIcon(
      icon: AnimatedIcons.menu_arrow,
      progress: transitionAnimation,
    ),
    onPressed: () {
      close(context, <span class="hljs-keyword">null</span>);
    },
  );
}
</code></pre>
<p data-nodeid="47227">关闭当前页面使用 close(context, null) 即可实现。</p>
<p data-nodeid="47228">buildResults 为搜索结果显示列表，buildSuggestions 为搜索提示列表，在这里我们返回一个空 ListView() 就行。</p>
<p data-nodeid="47229">在上面基础上，我们需要修改默认的搜索框的提示，并且需要匹配当前主题的颜色字体等，需要做以下两部分逻辑。</p>
<pre class="lang-dart" data-nodeid="47230"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">修改提示框内容</span></span>
<span class="hljs-built_in">String</span> <span class="hljs-keyword">get</span> searchFieldLabel =&gt; <span class="hljs-string">'用户、帖子'</span>;
<span class="hljs-meta">@override</span>
ThemeData appBarTheme(BuildContext context) {
  <span class="hljs-keyword">final</span> ThemeData theme = Theme.of(context);
  <span class="hljs-keyword">return</span> theme.copyWith(
      inputDecorationTheme: InputDecorationTheme(),
      primaryColor: theme.primaryColor,
      primaryIconTheme: theme.primaryIconTheme,
      primaryColorBrightness: theme.primaryColorBrightness,
      primaryTextTheme: theme.primaryTextTheme
  );
}
</code></pre>
<p data-nodeid="50691" class="">上面代码中第 2 行是修改默认搜索框提示，第 5 至第 17 行则是匹配当前应用主题。完整代码可<a href="https://github.com/love-flutter/flutter-column" data-nodeid="50695">参考 github 源码。</a></p>

<h3 data-nodeid="47232">总结</h3>
<p data-nodeid="47233" class="">本课时介绍了控件 Scaffold 的一些基础用法，着重介绍了其中三个比较常用的属性 bottomNavigationBar、appBar 和 drawer，同时使用这些属性完成了我们顶部导航栏、底部导航栏、侧边导航栏和搜索功能的实现。学完本课时你需要掌握这些基础的导航栏设计的使用方法，其次了解控件 Scaffold 的其他属性的用法。</p>
<p data-nodeid="50385" class="">本课时实现了 App 的基础结构，下一课时我将从内容展示的多样式来实现具体的 App 页面内容。</p>











<p data-nodeid="47235" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="47324">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### **华：
> 老师，我运行本节课的源码报错，请问是什么原因** BUILD FAILED **Xcode's output:↳ lib/pages/entrance_top_bar.dart:10:1: Error: 'Router' is imported from both 'package:flutter/src/widgets/router.dart' and 'package:two_you_friend/router.dart'.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 原因是 flutter 在更新版本以后，把 router 进行调整，增加进原生库了，因此产生了冲突，近几天内，我会重新更新一版本代码，适应最新版本的flutter。

##### **辉：
> 中间 吸顶效果怎么做

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 参考这里的代码，第一部分就是吸顶效果的实现。https://api.flutter.dev/flutter/widgets/NestedScrollView-class.html

##### **龙：
> 可以同时实现顶部导航+底部导航+侧边栏么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以的，你可以直接把源代码拿出来，把三部分逻辑都打开。没有任何问题的哈。

