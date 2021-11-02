<p data-nodeid="29698" class="">你好，欢迎来到第 13 课时，上一课时我们实现了多种列表样式，但是缺乏下拉刷新和上拉加载更多的功能，本课时就来完善这部分的功能。实现下拉刷新和上拉加载更多，需要应用到 RefreshIndicator 组件 ，接下来我们就先了解这个组件的一些作用。</p>
<h3 data-nodeid="29699">RefreshIndicator</h3>
<p data-nodeid="29700">该组件主要的作用是在下拉时实现刷新，具体看下组件参数的一些作用。</p>
<pre class="lang-dart" data-nodeid="29701"><code data-language="dart">RefreshIndicator({
  Key key,
  <span class="hljs-meta">@required</span> <span class="hljs-keyword">this</span>.child, <span class="hljs-comment">// 子组件，需要更新的组件列表</span>
  <span class="hljs-keyword">this</span>.displacement = <span class="hljs-number">40.0</span>, <span class="hljs-comment">// 刷新指示器离顶部的位置</span>
  <span class="hljs-meta">@required</span> <span class="hljs-keyword">this</span>.onRefresh, <span class="hljs-comment">// 下拉触发函数，该函数必须是 Future&lt;void&gt;</span>
  <span class="hljs-keyword">this</span>.color, <span class="hljs-comment">// 设置指示器的颜色</span>
  <span class="hljs-keyword">this</span>.backgroundColor, <span class="hljs-comment">// 设置指示器的颜色</span>
  <span class="hljs-keyword">this</span>.notificationPredicate = defaultScrollNotificationPredicate, <span class="hljs-comment">// 检查组件是否需要监听下拉事件</span>
  <span class="hljs-keyword">this</span>.semanticsLabel, <span class="hljs-comment">// 设置指示器的一个标签名字</span>
  <span class="hljs-keyword">this</span>.semanticsValue, <span class="hljs-comment">// 加载进度，一般使用百分比</span>
  <span class="hljs-keyword">this</span>.strokeWidth = <span class="hljs-number">2.0</span> <span class="hljs-comment">// 指示器的像素大小，默认 2.0</span>
})
</code></pre>
<p data-nodeid="29702">了解完 RefreshIndicator 组件，我们再来看下本课时所要实现的一个效果，如图 1 动画所示。</p>
<p data-nodeid="29703"><img src="https://s0.lgstatic.com/i/image/M00/36/B5/CgqCHl8X9e6AN6lyAH6AtsDFbHo367.gif" alt="20200711_155132.gif" data-nodeid="29791"><br>
图 1 下拉刷新上拉加载效果</p>
<p data-nodeid="29704">图 1 中的效果包含了下拉刷新，上拉滑动翻页，在翻页到最后一页时，给了一定的提示信息。接下来我们就实现这个功能。</p>
<h3 data-nodeid="29705">实现原理</h3>
<p data-nodeid="29706">前面我介绍到了下拉刷新功能，使用到的是 RefreshIndicator 组件。而上拉加载更多，使用的是上课时 ListView.separated 中的 controller 属性，通过监听上拉动作，来判断是否需要执行下一页翻页。</p>
<p data-nodeid="29707">那么在实现代码前，我们还是需要做一些前期的准备。因为现在涉及了翻页，因此 API 返回的数据结构不仅仅是数据列表了，需要加上翻页相关的一些参数，具体我们来看下实现方案。</p>
<h4 data-nodeid="29708">API 增加返回结构</h4>
<p data-nodeid="29709">API 的数据结构调整为下面的一个 JSON 格式。</p>
<pre class="lang-json" data-nodeid="29710"><code data-language="json">{
   <span class="hljs-attr">"ret"</span> : <span class="hljs-number">0</span>, 
   <span class="hljs-attr">"message"</span> : <span class="hljs-string">"success"</span>, 
   <span class="hljs-attr">"hasMore"</span> : <span class="hljs-literal">true</span>,
   <span class="hljs-attr">"lastId"</span> : <span class="hljs-literal">null</span>,
}
</code></pre>
<ul data-nodeid="29711">
<li data-nodeid="29712">
<p data-nodeid="29713">ret，表示返回的状态码，0 表示成功。</p>
</li>
<li data-nodeid="29714">
<p data-nodeid="29715">message，返回的提示信息。</p>
</li>
<li data-nodeid="29716">
<p data-nodeid="29717">hasMore，表示是否还有更多，或者说下一页。</p>
</li>
<li data-nodeid="29718">
<p data-nodeid="29719">lastId，翻页标识。</p>
</li>
</ul>
<p data-nodeid="29720">根据如上的结构，我们需要去 Struct 中新建一个 api_ret_info.dart 用来保存所有相关的 api 返回结构，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="29721"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/content_detail.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">api 拉取content list返回结构</span></span>
<span class="hljs-comment">///
<span class="markdown">/// {</span></span>
<span class="hljs-comment">///   <span class="markdown">"ret" : 0,</span></span>
<span class="hljs-comment">///   <span class="markdown">"message" : "success",</span></span>
<span class="hljs-comment">///   <span class="markdown">"hasMore" : true,</span></span>
<span class="hljs-comment">///   <span class="markdown">"lastId" : null,</span></span>
<span class="hljs-comment">/// <span class="markdown">}</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StructApiContentListRetInfo</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">用户的昵称</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">int</span> ret;
  <span class="hljs-comment">/// <span class="markdown">用户头像信息</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> message;
  <span class="hljs-comment">/// <span class="markdown">是否还有更多</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">bool</span> hasMore;
  <span class="hljs-comment">/// <span class="markdown">最后一个id</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> lastId;
  <span class="hljs-comment">/// <span class="markdown">具体的content list</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; data;
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  <span class="hljs-keyword">const</span> StructApiContentListRetInfo(
      <span class="hljs-keyword">this</span>.ret, <span class="hljs-keyword">this</span>.message, <span class="hljs-keyword">this</span>.hasMore, <span class="hljs-keyword">this</span>.lastId, <span class="hljs-keyword">this</span>.data
      );
}
</code></pre>
<p data-nodeid="29722">上面代码已经是一个比较常见的 Struct 结构。完成 Struct 代码后，我们再来修改 API 文件，将接口返回的类型从原来的 List 修改为 StructApiContentListRetInfo ，其次在函数中增加参数，用来判断是否为下一页，具体代码实现如下：</p>
<pre class="lang-dart" data-nodeid="29723"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">拉取用户内容推荐帖子列表</span></span>
StructApiContentListRetInfo getRecommendList([lastId = <span class="hljs-keyword">null</span>]) {
  <span class="hljs-keyword">if</span>(lastId != <span class="hljs-keyword">null</span>) {
    <span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; dataList = [
      StructContentDetail(),
      ...
    ];
    <span class="hljs-keyword">return</span> StructApiContentListRetInfo(
      <span class="hljs-number">0</span>,
      <span class="hljs-string">'success'</span>,
      <span class="hljs-keyword">false</span>,
      <span class="hljs-string">'2001'</span>,
      dataList
    );
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; dataList =  [
      StructContentDetail()，
      ...
    ];
    <span class="hljs-keyword">return</span> StructApiContentListRetInfo(
        <span class="hljs-number">0</span>,
        <span class="hljs-string">'success'</span>,
        <span class="hljs-keyword">true</span>,
        <span class="hljs-string">'1010'</span>,
        dataList
    );
  }
}
</code></pre>
<p data-nodeid="29724">代码第 2 行修改了 API 接口返回的数据类型为 StructApiContentListRetInfo 。代码第 3 行，判断是否为下一页，如果是则返回下一页的数据，如果不是则返回第一页数据。最后返回的数据结构都是使用 StructApiContentListRetInfo 进行封装处理。</p>
<p data-nodeid="29725">完成上面前期准备后，我们再来修改核心主页面的一个逻辑处理，分为下拉刷新和上拉加载更多两部分。Two You APP 的首页会以内容+缩略图的展示方式，本课时也只从这部分来介绍。</p>
<h4 data-nodeid="29726">新增状态</h4>
<p data-nodeid="29727">为了实现该功能，我们需要新增以下几个状态变量：</p>
<pre class="lang-dart" data-nodeid="29728"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">首页推荐贴子列表</span></span>
<span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; contentList;
<span class="hljs-comment">/// <span class="markdown">列表事件监听</span></span>
ScrollController scrollController = ScrollController();
<span class="hljs-comment">/// <span class="markdown">是否存在下一页</span></span>
<span class="hljs-built_in">bool</span> hasMore;
<span class="hljs-comment">/// <span class="markdown">页面是否正在加载</span></span>
<span class="hljs-built_in">bool</span> isLoading;
<span class="hljs-comment">/// <span class="markdown">最后一个数据 ID</span></span>
<span class="hljs-built_in">String</span> lastId;
</code></pre>
<ul data-nodeid="29729">
<li data-nodeid="29730">
<p data-nodeid="29731">contentList，我们已经介绍过，是保存当前需要的列表元素；</p>
</li>
<li data-nodeid="29732">
<p data-nodeid="29733">scrollController，用来监听列表事件，主要是判断上拉加载更多功能；</p>
</li>
<li data-nodeid="29734">
<p data-nodeid="29735">hasMore，标记是否还存在下一页；</p>
</li>
<li data-nodeid="29736">
<p data-nodeid="29737">isLoading，用于标识页面是否正在加载中；</p>
</li>
<li data-nodeid="29738">
<p data-nodeid="29739">lastId，记录最后一条数据的 ID，用于翻页。</p>
</li>
</ul>
<h4 data-nodeid="29740">首次加载和下拉刷新</h4>
<p data-nodeid="29741">首次加载和下拉刷新本质上是一样的作用，因此我们将两个功能合并为一个函数 setFirstPage ，先来看下这个函数的实现。</p>
<pre class="lang-dart" data-nodeid="29742"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">处理首次拉取和刷新数据获取动作</span></span>
<span class="hljs-keyword">void</span> setFirstPage() {
  StructApiContentListRetInfo retInfo =
    ApiContentIndex().getRecommendList();
  setState(() {
    contentList = retInfo.data;
    hasMore = retInfo.hasMore;
    isLoading = <span class="hljs-keyword">false</span>;
    lastId = retInfo.lastId;
  });
}
</code></pre>
<p data-nodeid="29743">在上述代码中，首先获取 API 返回数据，然后将返回的数据初始化相应的状态属性。实现完 setFirstPage ，我们再补充调用该函数的两个逻辑，initState 和 onRefresh ，代码如下：</p>
<pre class="lang-dart" data-nodeid="29744"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">处理刷新操作</span></span>
Future onRefresh() {
  <span class="hljs-keyword">return</span> Future.delayed(<span class="hljs-built_in">Duration</span>(seconds: <span class="hljs-number">1</span>), () {
    setFirstPage();
  });
}
<span class="hljs-meta">@override</span>
<span class="hljs-keyword">void</span> initState() {
  <span class="hljs-keyword">super</span>.initState();
  <span class="hljs-comment">/// <span class="markdown">拉取首页接口数据</span></span>
  setFirstPage();
}
</code></pre>
<p data-nodeid="29745">完成以上两部分后，我们使用刚刚介绍的 RefreshIndicator 来实现下拉刷新，修改 build 逻辑，代码如下：</p>
<pre class="lang-dart" data-nodeid="29746"><code data-language="dart"><span class="hljs-meta">@override</span>
Widget build(BuildContext context) {
  <span class="hljs-keyword">return</span> RefreshIndicator(
    onRefresh: onRefresh, <span class="hljs-comment">// 调用刷新事件</span>
    child: ListView.separated(
      scrollDirection: Axis.vertical,
      shrinkWrap: <span class="hljs-keyword">true</span>,
      itemCount: contentList.length,
      itemBuilder: (BuildContext context, <span class="hljs-built_in">int</span> position) {
        <span class="hljs-keyword">return</span> ArticleCard(articleInfo: contentList[position]);
      },
      separatorBuilder: (context, index) {
        <span class="hljs-keyword">return</span> Divider(
          height: <span class="hljs-number">.5</span>,
          <span class="hljs-comment">//indent: 75,</span>
          color: Color(<span class="hljs-number">0xFFDDDDDD</span>),
        );
      },
    ),
  );
}
</code></pre>
<p data-nodeid="29747">在上面代码的第 3 行，使用到了 RefreshIndicator 组件，在第 4 行设置 onRefresh 调用 onRefresh 函数，这样就完成了下拉刷新功能。</p>
<h4 data-nodeid="29748">上拉加载更多</h4>
<p data-nodeid="29749">上拉加载更多使用到 ListView.separated 的 controller 属性功能，在 initState 中，我们首先要设置该滑动的监听事件，代码如下：</p>
<pre class="lang-dart" data-nodeid="29750"><code data-language="dart"><span class="hljs-meta">@override</span>
<span class="hljs-keyword">void</span> initState() {
  <span class="hljs-keyword">super</span>.initState();
  <span class="hljs-comment">/// <span class="markdown">拉取首页接口数据</span></span>
  setFirstPage();
  <span class="hljs-comment">/// <span class="markdown">监听上滑事件，活动加载更多</span></span>
  <span class="hljs-keyword">this</span>.scrollController.addListener(() {
    <span class="hljs-keyword">if</span>(!hasMore){
      <span class="hljs-keyword">return</span>;
    }
    <span class="hljs-keyword">if</span> (
    !isLoading &amp;&amp;
        scrollController.position.pixels &gt;=
            scrollController.position.maxScrollExtent
    ) {
      isLoading = <span class="hljs-keyword">true</span>;
      loadMoreData();
    }
  });
}
</code></pre>
<p data-nodeid="29751">上述代码的第 8 行到第 22 行，先判断是否还有更多内容，如果没有则下拉不处理任何事件，如果有更多内容，并且界面内容没有在加载中，则处理内容加载。第 13 行中的 isLoading 目的就是避免上一次内容未加载完，又继续请求触发。第 14 行作用是判断当前滚动位置是否大于等于最大滚动长度，大于则表示需要加载更多。触发加载时，首选需要将 isLoading 修改为 true，这里可以不用 setState ，避免因为使用 setState 而引发 build 逻辑。最后调用 loadMoreData 方法，我们来看下该方法的实现。</p>
<pre class="lang-dart" data-nodeid="29752"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">加载下一页</span></span>
<span class="hljs-keyword">void</span> loadMoreData() {
  StructApiContentListRetInfo retInfo =
    ApiContentIndex().getRecommendList(lastId);
  <span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; newList = retInfo.data;
  setState(() {
    isLoading = <span class="hljs-keyword">false</span>;
    hasMore = retInfo.hasMore;
    contentList.addAll(newList);
  });
}
</code></pre>
<p data-nodeid="29753">代码第 4 到第 6 行还是拉取 API 接口数据，获取到接口返回后，再使用 setState 来更新状态变量 contentList ，从而触发界面 build 。</p>
<p data-nodeid="29754">以上就完成了下拉刷新和上拉加载更多，但是这里还存在一些问题，比如在网络较慢的情况下，如何处理一些加载动作，或者在接口出错的情况下，如何友好地提示用户。下面我们就来完成这部分的一个简单优化。</p>
<h3 data-nodeid="29755">功能优化</h3>
<p data-nodeid="31738" class="">在 App 运行中，都没办法 100% 保证正确性，因此需要考虑到，在接口访问较慢或者接口报错的情况下，我们需要友好地提示用户。这里我们可以在接口报错或者其他异常时，显示通用报错组件。在接口请求较慢，或者加载到最后一页时，可以设计一个通用的 loading 组件来优化这部分体验。接下来我们就来实现这两个通用的组件。</p>







<h4 data-nodeid="29757">增加错误处理</h4>
<p data-nodeid="29758">首先我们需要创建一个比较通用的错误提示组件，该组件因为是一个通用组件，所以在 widgets/common 文件夹下创建 error.dart 。创建该组件的一些外部参数，代码如下：</p>
<pre class="lang-dart" data-nodeid="29759"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">具体的错误码</span></span>
<span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> errorCode;
<span class="hljs-comment">/// <span class="markdown">可点击的回调函数</span></span>
<span class="hljs-keyword">final</span> <span class="hljs-built_in">Function</span> action;
<span class="hljs-comment">/// <span class="markdown">默认构造函数</span></span>
<span class="hljs-keyword">const</span> CommonError({Key key, <span class="hljs-keyword">this</span>.errorCode, <span class="hljs-keyword">this</span>.action})
    :<span class="hljs-keyword">super</span>(key: key);
<span class="hljs-comment">/// <span class="markdown">返回提示信息</span></span>
<span class="hljs-keyword">static</span> <span class="hljs-keyword">const</span> errorMapping = {
  <span class="hljs-string">'server_error'</span> : <span class="hljs-string">'服务器响应错误'</span>,
  <span class="hljs-string">'error'</span> : <span class="hljs-string">'系统异常'</span>,
  <span class="hljs-string">'default'</span> : <span class="hljs-string">'服务异常'</span>
};
</code></pre>
<p data-nodeid="29760">主要看下 build 逻辑，build 中包含两部分，第一部分是解析页面跳转的数据，代码如下：</p>
<pre class="lang-dart" data-nodeid="29761"><code data-language="dart"><span class="hljs-built_in">String</span> newErrorCode = errorCode;
<span class="hljs-built_in">Function</span> newAction = action;
<span class="hljs-comment">// 获取来自接口 router 跳转参数</span>
<span class="hljs-keyword">if</span> (ModalRoute.of(context).settings.arguments != <span class="hljs-keyword">null</span>) {
  <span class="hljs-built_in">Map</span> dataInfo =
  JsonConfig.objectToMap(ModalRoute.of(context).settings.arguments);
  <span class="hljs-keyword">if</span>(dataInfo[<span class="hljs-string">'errorCode'</span>] != <span class="hljs-keyword">null</span>) {
    newErrorCode = dataInfo[<span class="hljs-string">'errorCode'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">String</span>;
  }
  <span class="hljs-keyword">if</span>(dataInfo[<span class="hljs-string">'action'</span>] != <span class="hljs-keyword">null</span>) {
    newAction = dataInfo[<span class="hljs-string">'action'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">Function</span>;
  }
}
<span class="hljs-comment">// 判断是否存在，不存在使用默认提示</span>
<span class="hljs-keyword">if</span>(errorMapping[newErrorCode] == <span class="hljs-keyword">null</span>) {
  newErrorCode = <span class="hljs-string">'default'</span>;
}
</code></pre>
<p data-nodeid="29762">上述代码中的第 4 到 13 行主要是为了处理有些来自 router 跳转的参数，router 跳转的参数是通过 ModalRoute.of(context).settings.arguments 方法来获取，这点在第 10 课时已经讲解过。第 15 行则是避免没有匹配到任何错误提示，给一个默认数据。</p>
<p data-nodeid="29763">以上是参数处理部分，接下来我们看下组件部分代码。</p>
<pre class="lang-dart" data-nodeid="29764"><code data-language="dart"><span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Scaffold(
  body: Container(
    alignment: Alignment.center,
    child: Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: &lt;Widget&gt;[
        Text(<span class="hljs-string">'<span class="hljs-subst">${errorMapping[newErrorCode]}</span>'</span>),
        RaisedButton(
          color: Colors.lightBlueAccent,
          highlightColor: Colors.lightBlueAccent[<span class="hljs-number">700</span>],
          colorBrightness: Brightness.dark,
          splashColor: Colors.lightBlueAccent,
          child: Icon(Icons.refresh),
          shape:RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(<span class="hljs-number">20.0</span>)
          ),
          onPressed: () {
            <span class="hljs-keyword">if</span>(newAction != <span class="hljs-keyword">null</span>) {
              newAction(); <span class="hljs-comment">// 处理重试</span>
            }
          },
        ),
      ],
    ),
  ),
);
</code></pre>
<p data-nodeid="29765">上面代码使用了按钮组件 RaisedButton ，这和我们之前所使用的 FaltButton 功能相似，都是一个操作按钮，点击触发效果的功能。不过两者在界面上有一定区别，前者是凸起的按钮，后者是扁平的按钮。为了能够让用户手动重试，我们按钮增加了 onPressed 属性，该属性在点击后会触发 newAction 函数的执行。</p>
<p data-nodeid="29766">接下来我们看下主页面是如何来调用的。首先在接口失败后，需要将一个状态变量 error 设置为 true ，其次在 build 前进行该字段的判断，当为 true 时，则显示错误页面，具体代码如下。</p>
<pre class="lang-dart" data-nodeid="29767"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">处理首次拉取和刷新数据获取动作</span></span>
<span class="hljs-keyword">void</span> setFirstPage() {
  StructApiContentListRetInfo retInfo =
  ApiContentIndex().getRecommendList();
  setState(() {
    <span class="hljs-keyword">if</span>(retInfo.ret != <span class="hljs-number">0</span>){ <span class="hljs-comment">// 判断返回是否正确</span>
      error = <span class="hljs-keyword">true</span>;
      <span class="hljs-keyword">return</span>;
    }
    error = <span class="hljs-keyword">false</span>;
    contentList = retInfo.data;
    hasMore = retInfo.hasMore;
    isLoading = <span class="hljs-keyword">false</span>;
    lastId = retInfo.lastId;
  });
}
</code></pre>
<p data-nodeid="29768">上述代码和前面介绍的 setFirstPage 的共同点是对接口进行了返回判断，报错则直接设置 error 状态即可。</p>
<pre class="lang-dart" data-nodeid="29769"><code data-language="dart"><span class="hljs-meta">@override</span>
Widget build(BuildContext context) {
  <span class="hljs-keyword">if</span>(error){
    <span class="hljs-keyword">return</span> CommonError(action: <span class="hljs-keyword">this</span>.setFirstPage);
  }
</code></pre>
<p data-nodeid="29770">build 部分代码我们只看前三行即可，当接口报错，则直接返回错误组件，并且传入重试函数<br>
setFirstPage 。以上就完成了对错误的体验优化，这部分也可以使用弹窗提示模式。</p>
<h4 data-nodeid="29771">增加 loading 效果</h4>
<p data-nodeid="29772">这里和上面错误处理一样，我们先还是创建一个通用的 loading 组件。在 widgets/common 下创建 loading.dart 文件。loading 组件会有三种状态：加载中、上拉加载提示、加载完成。将以上三种状态分别设计为三个小组件，我们主要看下核心组件的代码：</p>
<pre class="lang-dart" data-nodeid="29773"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">自行展示，load more还是已加载完成</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CommonLoadingButton</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">加载状态</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">bool</span> loadingState;
  <span class="hljs-comment">/// <span class="markdown">是否有更多</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">bool</span> hasMore;
  <span class="hljs-comment">/// <span class="markdown">默认构造函数</span></span>
  <span class="hljs-keyword">const</span> CommonLoadingButton({Key key, <span class="hljs-keyword">this</span>.loadingState, <span class="hljs-keyword">this</span>.hasMore}) :
        <span class="hljs-keyword">super</span>(key: key);
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">if</span>(!<span class="hljs-keyword">this</span>.hasMore) {
      <span class="hljs-keyword">return</span> NoMore();
    }
    <span class="hljs-keyword">if</span>(<span class="hljs-keyword">this</span>.loadingState) {
      <span class="hljs-keyword">return</span> Loading();
    } <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">return</span> LoadingStatic();
    }
  }
}
</code></pre>
<p data-nodeid="29774">上述代码第 14 行判断是否存在更多，没有则显示 NoMore 组件，再根据 loadingState 判断是显示加载中还是显示上拉加载提示。</p>
<p data-nodeid="29775">最后我们再来看下在 build 逻辑中是如何应用该组件的，代码如下：</p>
<pre class="lang-dart" data-nodeid="29776"><code data-language="dart">Widget build(BuildContext context) {
  <span class="hljs-keyword">if</span>(error){
    <span class="hljs-keyword">return</span> CommonError(action: <span class="hljs-keyword">this</span>.setFirstPage);
  }
  <span class="hljs-keyword">return</span> RefreshIndicator(
    onRefresh: onRefresh, <span class="hljs-comment">// 调用刷新事件</span>
    child: ListView.separated(
      scrollDirection: Axis.vertical,
      controller: scrollController,
      shrinkWrap: <span class="hljs-keyword">true</span>,
      itemCount: contentList.length + <span class="hljs-number">1</span>,
      itemBuilder: (BuildContext context, <span class="hljs-built_in">int</span> position) {
        <span class="hljs-keyword">if</span>(position &lt; <span class="hljs-keyword">this</span>.contentList.length) {
          <span class="hljs-keyword">return</span> ArticleCard(articleInfo: contentList[position]);
        }
        <span class="hljs-keyword">return</span> CommonLoadingButton(
            loadingState: isLoading, hasMore: hasMore
        );
      },
      separatorBuilder: (context, index) {
        <span class="hljs-keyword">return</span> Divider(
          height: <span class="hljs-number">.5</span>,
          <span class="hljs-comment">//indent: 75,</span>
          color: Color(<span class="hljs-number">0xFFDDDDDD</span>),
        );
      },
    ),
  );
}
</code></pre>
<p data-nodeid="29777">上述代码中的第 11 行，需要将原来的列表数量增加一个，主要是预留给 loading 组件，其次在第 13 行判断是否大于当前列表元素，小于则 build 贴子组件，大于等于则显示 loading 组件。</p>
<p data-nodeid="29778">以上就完成了错误处理和 loading 效果的优化，这两个功能也是通用组件，后续也会被使用到其他页面逻辑中。</p>
<h3 data-nodeid="29779">总结</h3>
<p data-nodeid="34564" class="">本课时介绍了刷新组件 RefreshIndicator 的一些基础属性，其次实践开发了下拉刷新和上拉加载更多功能，最后在页面基础上增加了一些优化体验的功能。学完本课时要掌握下拉刷新和上拉加载的知识，并且能够实践应用，其次需要了解通用组件的设计方法。</p>

<p data-nodeid="34878" class="">下一课时我们将在 App 基础上完成个人页面，并且在个人页面增加红点提醒功能，丰富 App 整体功能。</p>









<p data-nodeid="29782" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="29853">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### **涛：
> 挺好, 挺好.

