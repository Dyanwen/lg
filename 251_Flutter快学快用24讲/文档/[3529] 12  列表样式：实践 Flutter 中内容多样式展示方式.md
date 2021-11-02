<p data-nodeid="17459" class="">本课时，我将在导航栏基础上，设计一个 APP 首页推荐列表，以此来讲解 Flutter 中内容多样式的展示方式。</p>
<p data-nodeid="17460">列表的多样式包含内容+缩略图、图片九宫格以及单图信息流。接下来我将逐一讲解这三种类型的设计和实现原理。</p>
<h3 data-nodeid="17461">前期准备</h3>
<p data-nodeid="17462">本课时中的列表多样式会涉及 Flutter 控件 ListView ，该控件包含了多个构造函数，比如：默认构造函数、builder、separated 和 custom。</p>
<h4 data-nodeid="17463">ListView</h4>
<p data-nodeid="17464">ListView 下四种构造函数的使用场景都不相同，比如：</p>
<ul data-nodeid="17465">
<li data-nodeid="17466">
<p data-nodeid="17467">ListView 默认构造函数，适用于有限的小列表内容展示，一次性创建所有项目；</p>
</li>
<li data-nodeid="17468">
<p data-nodeid="17469">ListView.builder 构造函数用于处理包含大量数据的列表，其次它会在列表项滚动到屏幕上时创建该列表项；</p>
</li>
<li data-nodeid="17470">
<p data-nodeid="17471">ListView.separated 相比 ListView.builder 多了一个分隔符，其次更适用于固定项列表；</p>
</li>
<li data-nodeid="17472">
<p data-nodeid="17473">ListView.custom 可以自定义列表结构，使用场景不多，但是 ListView.builder 与 ListView.separated 都是基于 ListView.custom 来实现的。</p>
</li>
</ul>
<p data-nodeid="17689" class="">本课时因为是一个固定有限的列表，更适用于 ListView.separated ，因此本课时基于 ListView.separated 来讲解，这里我介绍下该控件的参数列表。</p>

<pre class="lang-dart" data-nodeid="17475"><code data-language="dart">ListView.separated({
  Key key,
  Axis scrollDirection = Axis.vertical, <span class="hljs-comment">// 滑动方向，垂直</span>
  <span class="hljs-built_in">bool</span> reverse = <span class="hljs-keyword">false</span>, <span class="hljs-comment">// 是否倒序</span>
  ScrollController controller, <span class="hljs-comment">// 控制滚动和监听滚动事件变化</span>
  <span class="hljs-built_in">bool</span> primary, <span class="hljs-comment">// false内容不足不滚动，true一直可滚动</span>
  ScrollPhysics physics, <span class="hljs-comment">// 列表滚动方式设置</span>
  <span class="hljs-built_in">bool</span> shrinkWrap = <span class="hljs-keyword">false</span>, <span class="hljs-comment">// item高度适配</span>
  EdgeInsetsGeometry padding, <span class="hljs-comment">// padding设置</span>
  <span class="hljs-meta">@required</span> IndexedWidgetBuilder itemBuilder, <span class="hljs-comment">// 设置列表内容</span>
  <span class="hljs-meta">@required</span> IndexedWidgetBuilder separatorBuilder, <span class="hljs-comment">// 设置分割内容</span>
  <span class="hljs-meta">@required</span> <span class="hljs-built_in">int</span> itemCount, <span class="hljs-comment">// 总数量</span>
  <span class="hljs-built_in">bool</span> addAutomaticKeepAlives = <span class="hljs-keyword">true</span>,  <span class="hljs-comment">// 该属性表示是否将列表项（子组件）包裹在AutomaticKeepAlive&nbsp;组件中，主要是为了避免被垃圾回收</span>
  <span class="hljs-built_in">bool</span> addRepaintBoundaries = <span class="hljs-keyword">true</span>, <span class="hljs-comment">// 该属性表示是否将列表项（子组件）包裹在addRepaintBoundaries&nbsp;组件中，为了避免重绘</span>
  <span class="hljs-built_in">bool</span> addSemanticIndexes = <span class="hljs-keyword">true</span>, <span class="hljs-comment">// 该属性表示是否将列表项（子组件）包裹在addSemanticIndexes&nbsp;组件中，用来提供无障碍语义</span>
  <span class="hljs-built_in">double</span> cacheExtent, <span class="hljs-comment">// 设置预加载区域</span>
})
</code></pre>
<p data-nodeid="17476">根据以上的配置信息，我们设置了一个比较通用的配置，如下。</p>
<pre class="lang-dart" data-nodeid="17477"><code data-language="dart">ListView.separated(
  scrollDirection: Axis.vertical,
  shrinkWrap: <span class="hljs-keyword">true</span>,
  itemCount: contentList.length,
  itemBuilder: (BuildContext context, <span class="hljs-built_in">int</span> position) {
    <span class="hljs-keyword">return</span> (Widget);
  },
  separatorBuilder: (context, index) {
    <span class="hljs-keyword">return</span> Divider(
      height: <span class="hljs-number">.5</span>,
      <span class="hljs-comment">//indent: 75,</span>
      color: Color(<span class="hljs-number">0xFFDDDDDD</span>),
    );
  },
)
</code></pre>
<p data-nodeid="17478">上面代码配置中，scrollDirection 设置为垂直滚动，shrinkWrap 设置为高度适配，separatorBuilder 使用灰色线条分割每列数据。</p>
<p data-nodeid="17479">以上就是 ListView 控件的一些基本知识，介绍完基本的知识点后，我们还需要做一些编码方面的前期准备。由于该交友 APP，在列表展示的是推荐帖子，因此需要使用到相应的帖子内容相关的数据结构。根据交友 APP 的数据需要，我们设计交友帖子对应的数据结构模块 Struct 、相应获取推荐帖子内容的 API 接口以及需要状态共享的 Model。</p>
<h4 data-nodeid="17480">Struct</h4>
<p data-nodeid="17481">首页推荐的交友帖子数据，涉及三个具体内容：用户信息部分、交友帖子数据、帖子的评论信息。因此需要创建好三个对应的 Struct 文件。</p>
<p data-nodeid="17482">1.user_info.dart，对应为 StructUserInfo 类，其数据结果如下。</p>
<pre class="lang-dart" data-nodeid="17483"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">用户信息</span></span>
<span class="hljs-comment">///
<span class="markdown">/// {</span></span>
<span class="hljs-comment">///   <span class="markdown">"nickname" : "string",</span></span>
<span class="hljs-comment">///   <span class="markdown">"headerUrl" : "string",</span></span>
<span class="hljs-comment">///   <span class="markdown">"uid" : "string"</span></span>
<span class="hljs-comment">/// <span class="markdown">}</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StructUserInfo</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">标题</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> nickName;
  <span class="hljs-comment">/// <span class="markdown">简要</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> headerUrl;
  <span class="hljs-comment">/// <span class="markdown">主要内容</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> uid;
  <span class="hljs-comment">/// <span class="markdown">默认构造函数</span></span>
  <span class="hljs-keyword">const</span> StructUserInfo(
      <span class="hljs-keyword">this</span>.uid,
      <span class="hljs-keyword">this</span>.nickName,
      <span class="hljs-keyword">this</span>.headerUrl
      );
}
</code></pre>
<p data-nodeid="17484">2.content_detail.dart，对应为 StructContentDetail 类，其数据结构比较长，我们这里只是给个 JSON 的例子，代码如下。</p>
<pre class="lang-json" data-nodeid="17485"><code data-language="json">{
   <span class="hljs-attr">"id"</span> : <span class="hljs-string">"string"</span>,
   <span class="hljs-attr">"title"</span> : <span class="hljs-string">"string"</span>,
   <span class="hljs-attr">"summary"</span> : <span class="hljs-string">"string"</span>,
   <span class="hljs-attr">"detailInfo"</span> : <span class="hljs-string">"string"</span>,
   <span class="hljs-attr">"uid"</span> : <span class="hljs-string">"string"</span>,
   <span class="hljs-attr">"userInfo"</span> : <span class="hljs-string">"StructUserInfo"</span>,
   <span class="hljs-attr">"articleImage"</span> : <span class="hljs-string">"string"</span>,
   <span class="hljs-attr">"likeNum"</span> : <span class="hljs-string">"int"</span>,
   <span class="hljs-attr">"commentNum"</span> : <span class="hljs-string">"int"</span>
}
</code></pre>
<p data-nodeid="17486">3.comment_info.dart，对应为 StructCommentInfo 类，代码如下。</p>
<pre class="lang-dart" data-nodeid="17487"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/user_info.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">用户信息</span></span>
<span class="hljs-comment">///
<span class="markdown">/// {</span></span>
<span class="hljs-comment">///   <span class="markdown">"userInfo" : "StructUserInfo",</span></span>
<span class="hljs-comment">///   <span class="markdown">"comment" : "string"</span></span>
<span class="hljs-comment">/// <span class="markdown">}</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StructCommentInfo</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">用户的昵称</span></span>
  <span class="hljs-keyword">final</span> StructUserInfo userInfo;
  <span class="hljs-comment">/// <span class="markdown">用户头像信息</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> comment;
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  <span class="hljs-keyword">const</span> StructCommentInfo(<span class="hljs-keyword">this</span>.userInfo, <span class="hljs-keyword">this</span>.comment);
}
</code></pre>
<h4 data-nodeid="17488">API</h4>
<p data-nodeid="17489">有了以上基础的数据结构后，我们再来开发对应具体的 API，通过 API 部分拉取具体的首页推荐的帖子列表。在 API 文件夹中创建一个 content 文件夹，并且在 content 文件夹中创建 index.dart API 文件类，在该类中创建三个方法（这里使用的是假数据，未真正的调用服务端）。</p>
<p data-nodeid="17490">1.getOneById，根据内容 ID 拉取内容详情</p>
<pre class="lang-dart" data-nodeid="17491"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">根据内容id拉取内容详情</span></span>
StructContentDetail getOneById(<span class="hljs-built_in">String</span> id) {
   StructContentDetail detailInfo = StructContentDetail(
      <span class="hljs-string">'1001'</span>,
      <span class="hljs-string">'hello test'</span>,
      <span class="hljs-string">'summary'</span>,
      <span class="hljs-string">'detail info <span class="hljs-subst">${id}</span>'</span>,
      <span class="hljs-string">'1001'</span>,
      <span class="hljs-number">1</span>,
      <span class="hljs-number">2</span>,
      <span class="hljs-string">'https://i.pinimg.com/originals/e0/64/4b/e0644bd2f13db50d0ef6a4df5a756fd9.png'</span>
   );
   StructUserInfo userInfo = ApiUserInfoIndex.getOneById(detailInfo.uid);
   <span class="hljs-keyword">return</span> StructContentDetail(
       detailInfo.uid, detailInfo.title,
       detailInfo.summary, detailInfo.detailInfo,
       detailInfo.uid, detailInfo.likeNum,detailInfo.commentNum,
       detailInfo.articleImage, userInfo: userInfo
   );
}
</code></pre>
<p data-nodeid="17492">上面代码获取到初始的单条内容，然后基于用户信息的 API 补全用户信息部分，返回 StructContentDetail 数据结构。</p>
<p data-nodeid="17493">2.getRecommendList，获取首页推荐的内容列表</p>
<pre class="lang-dart" data-nodeid="17494"><code data-language="dart"><span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; getRecommendList() {
  <span class="hljs-keyword">return</span> [
   StructContentDetail(...),
   StructContentDetail(...),
  ]
}
</code></pre>
<p data-nodeid="18151">这部分代码就比较简单，获取具体的推荐内容列表，并返回 List<code data-backticks="1" data-nodeid="18154">&lt;StructContentDetail&gt;</code> 列表数据。</p>
<p data-nodeid="18152">3.getFollowList，获取关注人的内容列表</p>

<p data-nodeid="17496">这部分和 getRecommendList 方法实现完全一样，其中拉取的只是关注人的帖子列表。</p>
<h4 data-nodeid="17497">Model</h4>
<p data-nodeid="17498">这里只涉及我们第 07 课时所演示例子的知识点——应用 Provider 来实现状态管理。实现原理一样，唯一不同点是这里需要保存多个帖子的点赞数量，因此需要将这个状态变量设计为一个 Map，其次需要将 get 方法进行修改，使用帖子 id 作为键名。下面为部分代码，其他部分代码请查看 github 源码。</p>
<pre class="lang-dart" data-nodeid="17499"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">name状态管理模块</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LikeNumModel</span> <span class="hljs-title">with</span> <span class="hljs-title">ChangeNotifier</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">声明私有变量</span></span>
  <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">int</span>&gt; _likeInfo;
  <span class="hljs-comment">/// <span class="markdown">设置get方法</span></span>
  <span class="hljs-built_in">int</span> getLikeNum(<span class="hljs-built_in">String</span> articleId, [<span class="hljs-built_in">int</span> likeNum = <span class="hljs-number">0</span>]) {
    <span class="hljs-keyword">if</span>(_likeInfo == <span class="hljs-keyword">null</span>){
      _likeInfo = {};
    }
    <span class="hljs-keyword">if</span>(articleId == <span class="hljs-keyword">null</span>){
      <span class="hljs-keyword">return</span> likeNum;
    }
    <span class="hljs-keyword">if</span>(_likeInfo[articleId] == <span class="hljs-keyword">null</span>) {
      _likeInfo[articleId] = likeNum;
    }

    <span class="hljs-keyword">return</span> _likeInfo[articleId];
  }
}
</code></pre>
<p data-nodeid="17500">接下来我们就具体看下这三种列表样式的实现原理。</p>
<h3 data-nodeid="17501">内容+缩略图</h3>
<p data-nodeid="17502">这种样式的列表内容较为常见，每一条信息包含帖子的标题和简要信息，右侧为一个缩略图，底部栏为头像、点赞和评论相关内容，具体效果截图如下图 1。</p>
<p data-nodeid="17503"><img src="https://s0.lgstatic.com/i/image/M00/33/F0/CgqCHl8RIO-AVjCIAANJnR6nMPc935.png" alt="图片1.png" data-nodeid="17609"></p>
<h4 data-nodeid="17504">组件设计</h4>
<p data-nodeid="17505">按照我们 06 课时的知识点，我们需要将界面的组件进行拆解分析，由于这部分我们在 06 课时也分析过，因此这里比较快速地分析出下面的一个组件树，如图 2 所示。</p>
<p data-nodeid="17506"><img src="https://s0.lgstatic.com/i/image/M00/33/F0/CgqCHl8RIPyAE8_gAALBPW0-Bj0112.png" alt="图片2.png" data-nodeid="17614"></p>
<h4 data-nodeid="17507">实现原理</h4>
<p data-nodeid="17508">组件部分的实现逻辑，我们在 06 课时已经详细介绍过，这里就不一一细讲。接下来我们主要说下列表部分的实现，核心代码在 home_page/index.dart 中，首先还是 import 对应需要的库和组件库。</p>
<pre class="lang-dart" data-nodeid="17509"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/api/content/index.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/widgets/home_page/article_card.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/content_detail.dart'</span>;
</code></pre>
<p data-nodeid="17510">其中的 API 为我们拉取内容的接口， article_card 为我们每条展示的内容的组件， content_detail 则为 Struct 类。</p>
<p data-nodeid="17511">为了后续动态内容的需要，这里将该类设计为一个有状态类。</p>
<pre class="lang-dart" data-nodeid="17512"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">首页</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageIndex</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatefulWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  <span class="hljs-keyword">const</span> HomePageIndex({Key, key});
  <span class="hljs-meta">@override</span>
  createState() =&gt; HomePageIndexState();
}
<span class="hljs-comment">/// <span class="markdown">具体的state类</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageIndexState</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">State</span>&lt;<span class="hljs-title">HomePageIndex</span>&gt; </span>{
  <span class="hljs-comment">/// <span class="markdown">首页推荐帖子列表</span></span>
  <span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; contentList;
  <span class="hljs-meta">@override</span>
  <span class="hljs-keyword">void</span> initState() {
    <span class="hljs-keyword">super</span>.initState();
    <span class="hljs-comment">// 拉取推荐内容</span>
    setState(() {
      contentList = ApiContentIndex().getRecommendList();
    });
  }
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
  }
}
</code></pre>
<p data-nodeid="17513">上面代码中的第 20 行就是通过 API 去拉取具体的推荐内容列表，并使用 setState 来触发界面更新。接下来再看下 build 逻辑，在列表展示部分，我们需要使用到 ListView.separated 控件，下面看下这部分的核心代码。</p>
<pre class="lang-dart" data-nodeid="17514"><code data-language="dart"><span class="hljs-meta">@override</span>
Widget build(BuildContext context) {
  <span class="hljs-keyword">return</span> ListView.separated(
    scrollDirection: Axis.vertical,
    shrinkWrap: <span class="hljs-keyword">true</span>,
    itemCount: contentList.length,
    itemBuilder: (BuildContext context, <span class="hljs-built_in">int</span> position) {
      <span class="hljs-keyword">return</span> ArticleCard(articleInfo: <span class="hljs-keyword">this</span>.contentList[position]);
    },
    separatorBuilder: (context, index) {
      <span class="hljs-keyword">return</span> Divider(
        height: <span class="hljs-number">.5</span>,
        <span class="hljs-comment">//indent: 75,</span>
        color: Color(<span class="hljs-number">0xFFDDDDDD</span>),
      );
    },
  );
}
</code></pre>
<p data-nodeid="17515">其他部分与我们一开始介绍的 ListView.separated 标准部分相同，唯一不同的就是在 itemBuilder 做了组件的插入，这里针对每个数组元素所进行的操作，都是返回一个 article_card 组件。</p>
<p data-nodeid="17516">以上就完成了一个内容+缩略图组件的设计，接下来我们看下大图列表的设计。</p>
<h3 data-nodeid="17517">大图列表</h3>
<p data-nodeid="17518">大图列表是一个大小图穿插的功能，可以分为三个一行插入，奇数行显示大小图组合，偶数行显示三小图组合。其中在大小图组合中，大图位置随机为第一个或者最后一个，具体效果如图 3 所示。</p>
<p data-nodeid="17519"><img src="https://s0.lgstatic.com/i/image/M00/33/E5/Ciqc1F8RIRaAUc1mAAfclNZgPPA347.png" alt="图片3.png" data-nodeid="17634"></p>
<h4 data-nodeid="17520">组件设计</h4>
<p data-nodeid="17521">根据上面的规则，我们将三个图片分为一组，则存在 2 种组件组合规则，如图 4 所示的左右图的两个组合规则。图 4 左边为三小图并列组合规则，图 4 右侧为大小图组合规则，其次这部分还可能出现两种情况，第一种是第一个为大图，第二种是最后一个为大图，也就是“大小小”和“小小大”组件组合规则。</p>
<p data-nodeid="17522"><img src="https://s0.lgstatic.com/i/image/M00/33/E5/Ciqc1F8RISOAdHsLAAHLoId25k8599.png" alt="图片4.png" data-nodeid="17639"></p>
<h4 data-nodeid="17523">实现原理</h4>
<p data-nodeid="17524">我们创建 home_page/img_flow.dart 来表示这个大图列表组件的页面。然后为这个页面增加一个跳转入口，修改 11 课时中的左侧菜单栏文件 draw.dart ，将菜单名称修改为图片流，并且跳转到 tyfapp://imgflow 这个地址（这里需要去 router 中注册 imgflow 路由，注册方法如下代码的第 10 行）。</p>
<pre class="lang-dart" data-nodeid="17525"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">路由配置信息</span></span>
<span class="hljs-comment">/// <span class="markdown">widget 为组件</span></span>
<span class="hljs-comment">/// <span class="markdown">entranceIndex 为首页位置，如果非首页则为-1</span></span>
<span class="hljs-comment">/// <span class="markdown">params 为组件需要的参数数组</span></span>
<span class="hljs-keyword">const</span> <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, StructRouter&gt; routerMapping = {
  <span class="hljs-string">'homepage'</span>: StructRouter(HomePageIndex(), <span class="hljs-number">0</span>, <span class="hljs-keyword">null</span>),
  <span class="hljs-string">'userpage'</span>: StructRouter(UserPageIndex(), <span class="hljs-number">2</span>, [<span class="hljs-string">'userId'</span>]),
  <span class="hljs-string">'contentpage'</span>: StructRouter(ArticleDetailIndex(), <span class="hljs-number">-1</span>, [<span class="hljs-string">'articleId'</span>]),
  <span class="hljs-string">'default'</span>: StructRouter(HomePageIndex(), <span class="hljs-number">0</span>, <span class="hljs-keyword">null</span>),
  <span class="hljs-string">'imgflow'</span>: StructRouter(HomePageImgFlow(), <span class="hljs-number">-1</span>, <span class="hljs-keyword">null</span>),
  <span class="hljs-string">'singlepage'</span>: StructRouter(HomePageSingle(), <span class="hljs-number">-1</span>, <span class="hljs-keyword">null</span>)
};
</code></pre>
<p data-nodeid="17526">接下来实现 HomePageImgFlow 这个类，首先还是导入相应的组件库、 Struct 以及 API 接口，代码如下：</p>
<pre class="lang-dart" data-nodeid="17527"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/api/content/index.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/content_detail.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/widgets/home_page/img_card.dart'</span>;
</code></pre>
<p data-nodeid="17528">然后开始创建有状态类 HomePageImgFlow ，并在 initState 中拉取接口数据。</p>
<pre class="lang-dart" data-nodeid="17529"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">九宫格首页</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageImgFlow</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatefulWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  <span class="hljs-keyword">const</span> HomePageImgFlow({Key, key});
  <span class="hljs-meta">@override</span>
  createState() =&gt; HomePageImgFlowState();
}
<span class="hljs-comment">/// <span class="markdown">具体的state类</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageImgFlowState</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">State</span>&lt;<span class="hljs-title">HomePageImgFlow</span>&gt; </span>{
  <span class="hljs-comment">/// <span class="markdown">首页推荐帖子列表</span></span>
  <span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; contentList;
  <span class="hljs-meta">@override</span>
  <span class="hljs-keyword">void</span> initState() {
    <span class="hljs-keyword">super</span>.initState();
    <span class="hljs-comment">// 拉取推荐内容</span>
    setState(() {
      contentList = ApiContentIndex().getRecommendList();
    });
  }
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
  }
}
</code></pre>
<p data-nodeid="17530">上面代码和第一部分内容+缩略图的实现原理基本一致，在 build 方法中，除了 itemBuilder 逻辑不一样，其他实现均一样，所以我们主要看下 itemBuilder 代码。</p>
<pre class="lang-dart" data-nodeid="17531"><code data-language="dart"><span class="hljs-meta">@override</span>
Widget build(BuildContext context) {
  <span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; tmpList = [];
  <span class="hljs-keyword">return</span> ListView.separated(
    scrollDirection: Axis.vertical,
    shrinkWrap: <span class="hljs-keyword">true</span>,
    itemCount: contentList.length,
    itemBuilder: (BuildContext context, <span class="hljs-built_in">int</span> position) {
      <span class="hljs-keyword">if</span> (position % <span class="hljs-number">3</span> == <span class="hljs-number">0</span>) {
        <span class="hljs-comment">// 起始位置初始赋值</span>
        tmpList = [<span class="hljs-keyword">this</span>.contentList[position]];
      } <span class="hljs-keyword">else</span> {
        <span class="hljs-comment">// 非初始则插入列表</span>
        tmpList.add(<span class="hljs-keyword">this</span>.contentList[position]);
      }
      <span class="hljs-comment">// 判断数据插入时机，如果最后一组或者满足三个一组则插入</span>
      <span class="hljs-keyword">if</span> (position == contentList.length - <span class="hljs-number">1</span> || (position + <span class="hljs-number">1</span>) % <span class="hljs-number">3</span> == <span class="hljs-number">0</span>) {
        <span class="hljs-keyword">return</span> ImgCard(
            position: position,
            articleInfoList: tmpList,
            <span class="hljs-comment">// 确认是否为最后数据，最后数据无须处理大小图问题</span>
            isLast: position == contentList.length - <span class="hljs-number">1</span>);
      }
      <span class="hljs-keyword">return</span> Container();
    },
    separatorBuilder: (context, index) {
      <span class="hljs-keyword">return</span> Divider(
        height: <span class="hljs-number">.1</span>,
        <span class="hljs-comment">//indent: 75,</span>
        color: Color(<span class="hljs-number">0xFFDDDDDD</span>),
      );
    },
  );
}
</code></pre>
<p data-nodeid="17532">上面代码第 3 行，初始化定义了一个临时数组，该数组用来保存临时需要插入的列表，代码的第 10 行判断是否为 3 的倍数位置，例如第 0 、3 、6，对于这些位置需要将 tmpList 重新赋值，如果非这些位置，则往 tmpList 插入。</p>
<p data-nodeid="17533">代码第 19 行则判断 tmpList 是否满足 3 个，或者是否为最后一组，如果满足两个条件的任意一个则返回 ImgCard 组件，如果不是则返回一个空元素控件。</p>
<p data-nodeid="17534">接下来我们来看下 ImgCard 中的 build 代码。</p>
<pre class="lang-dart" data-nodeid="17535"><code data-language="dart"><span class="hljs-meta">@override</span>
Widget build(BuildContext context) {
  <span class="hljs-keyword">if</span> (isLast) {
    <span class="hljs-keyword">return</span> withSmallPic(context);
  }
  <span class="hljs-keyword">if</span> ((position + <span class="hljs-number">1</span>) % <span class="hljs-number">6</span> == <span class="hljs-number">3</span>) {
    <span class="hljs-keyword">return</span> withBigPic(context);
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-keyword">return</span> withSmallPic(context);
  }
}
</code></pre>
<p data-nodeid="17536">上面代码中的第 3 行是判断是否为最后一组，最后一组则使用小图模式，如果是奇数组则使用大图模式，是偶数组则使用小图模式。小图模式的实现比较简单，使用 flex 布局，并列显示三个即可。这里我们看下实现大图模式的代码。</p>
<pre class="lang-dart" data-nodeid="17537"><code data-language="dart"><span class="hljs-keyword">return</span> Row(
  children: &lt;Widget&gt;[
    Expanded(
      flex: <span class="hljs-number">6</span>,
      child: getFlatImg(context, articleInfoList[<span class="hljs-number">0</span>], <span class="hljs-number">200</span>),
    ),
    Expanded(
      flex: <span class="hljs-number">3</span>,
      child: Column(
        children: &lt;Widget&gt;[
          getFlatImg(context, articleInfoList[<span class="hljs-number">1</span>]),
          Padding(padding: EdgeInsets.only(top: <span class="hljs-number">2</span>)),
          getFlatImg(context, articleInfoList[<span class="hljs-number">2</span>]),
        ],
      ),
    ),
  ],
);
</code></pre>
<p data-nodeid="17538">这个组件布局也是使用 flex 来实现，大图占 6 小图占 3，其次小图使用 Column 控件来列表显示。<br>
以上就完成了大图列表的实现方式，接下来我们再看下单信息流模式。</p>
<h3 data-nodeid="17539">单信息流</h3>
<p data-nodeid="17540">单信息流模式有点类似于目前比较流行的短视频应用，在这里我们用简单的方式来介绍下实现原理。单信息流模式使用图片作为背景，右侧为头像、评论和点赞信息，最底部显示帖子的标题和摘要部分，效果如图 5 所示。</p>
<p data-nodeid="17541"><img src="https://s0.lgstatic.com/i/image/M00/33/E5/Ciqc1F8RIUWAJaWRAAQ7AhPuYYY687.png" alt="图片5.png" data-nodeid="17660"></p>
<h4 data-nodeid="17542">组件设计</h4>
<p data-nodeid="17543">根据图 5 的效果图，我们按照 06 课时的知识点，绘制出图 6 的一个组件树。</p>
<p data-nodeid="17544"><img src="https://s0.lgstatic.com/i/image/M00/33/E5/Ciqc1F8RIVKAZyj1AAJ_M5lRbOU489.png" alt="图片6.png" data-nodeid="17665"></p>
<p data-nodeid="17545">完成组件设计后，我们再根据组件树创建相应组件，以及实现相应组件代码。</p>
<h4 data-nodeid="17546">实现原理</h4>
<p data-nodeid="17547">我们创建 home_page/single.dart 来表示这个单信息流组件的页面，然后修改最开始的左侧菜单栏文件 draw.dart ，再修改第二个菜单为单图片信息，并且跳转到 tyfapp://singlepage 这个地址（这里需要去 router 中注册 singlepage 路由，具体注册的代码部分，在上面大图列表中已经说明）。</p>
<p data-nodeid="17548">接下来实现 HomePageSingle 这个类，首先还是导入相应的组件库、 Struct 以及 API 接口，代码如下：</p>
<pre class="lang-dart" data-nodeid="17549"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/api/content/index.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/widgets/home_page/single_bottom_summary.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/widgets/home_page/single_like_bar.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/widgets/home_page/single_right_bar.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/content_detail.dart'</span>;
</code></pre>
<p data-nodeid="17550">接下来创建有状态类组件，并且在 initState 中获取接口数据，并初始化赋值。</p>
<pre class="lang-dart" data-nodeid="17551"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">单个内容首页</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageSingle</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatefulWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  <span class="hljs-keyword">const</span> HomePageSingle({Key, key});
  <span class="hljs-meta">@override</span>
  createState() =&gt; HomePageSingleState();
}
<span class="hljs-comment">/// <span class="markdown">具体的state类</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePageSingleState</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">State</span>&lt;<span class="hljs-title">HomePageSingle</span>&gt; </span>{
  <span class="hljs-comment">/// <span class="markdown">index position</span></span>
  <span class="hljs-built_in">int</span> indexPos;
  <span class="hljs-comment">/// <span class="markdown">首页推荐帖子列表</span></span>
  <span class="hljs-built_in">List</span>&lt;StructContentDetail&gt; contentList;
  <span class="hljs-meta">@override</span>
  <span class="hljs-keyword">void</span> initState() {
    <span class="hljs-keyword">super</span>.initState();
    indexPos = <span class="hljs-number">0</span>;
    <span class="hljs-comment">// 拉取推荐内容</span>
    setState(() {
      contentList = ApiContentIndex().getRecommendList();
    });
  }
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
  }
</code></pre>
<p data-nodeid="17552">代码中的 indexPos 代表当前展示的内容位置，我们主要看下 build 逻辑的代码。</p>
<pre class="lang-dart" data-nodeid="17553"><code data-language="dart"><span class="hljs-keyword">return</span> Container(
  height: MediaQuery.of(context).size.height,
  width: MediaQuery.of(context).size.width,
  decoration: BoxDecoration(
      image: DecorationImage(
          image: NetworkImage(contentList[indexPos].articleImage),
          fit: BoxFit.contain)),
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.end,
    children: &lt;Widget&gt;[
      SingleRightBar(
          nickname: contentList[indexPos].userInfo.nickName,
          headerImage: contentList[indexPos].userInfo.headerUrl,
          commentNum: contentList[indexPos].commentNum),
      SingleLikeBar(
          articleId: contentList[indexPos].id,
          likeNum: contentList[indexPos].likeNum),
      SingleBottomSummary(
        articleId: contentList[indexPos].id,
        title: contentList[indexPos].title,
        summary: contentList[indexPos].summary,
      ),
    ],
  ),
);
</code></pre>
<p data-nodeid="17554">代码中的第 4 行就是为了设置背景图片，代码第 11 到第 22 行就是加载我们图 6 中的三个组件。三个组件中我们就只看 single_like_bar 组件的实现，其他两个组件实现原理较为简单，这个组件由于涉及状态管理，因此稍微复杂一些，所以着重说明下。接下来我们看下具体的逻辑实现过程。</p>
<p data-nodeid="17555">第一步导入相应的组件库和第三方库。</p>
<pre class="lang-dart" data-nodeid="17556"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:provider/provider.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/model/like_num_model.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/styles/text_syles.dart'</span>;
</code></pre>
<p data-nodeid="17557">第二步创建 SingleLikeBar 并且定义其初始化需要的参数，代码如下。</p>
<pre class="lang-dart" data-nodeid="17558"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">帖子文章的赞组件</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 包括点赞组件 icon ，以及组件点击效果</span></span>
<span class="hljs-comment">/// <span class="markdown">需要外部参数[likeNum],点赞数量</span></span>
<span class="hljs-comment">/// <span class="markdown">[articleId] 帖子的内容</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SingleLikeBar</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">帖子id</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> articleId;
  <span class="hljs-comment">/// <span class="markdown">like num</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">int</span> likeNum;
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  <span class="hljs-keyword">const</span> SingleLikeBar({Key key, <span class="hljs-keyword">this</span>.articleId, <span class="hljs-keyword">this</span>.likeNum})
      : <span class="hljs-keyword">super</span>(key: key);
  <span class="hljs-comment">/// <span class="markdown">返回组件信息</span></span>
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
  }
}
</code></pre>
<p data-nodeid="17559">最后看一下 build 逻辑的代码实现，基本和原来没有太大区别，只是在 Icon 和 Text 展示上从 Row 控件修改为 Column 控件。</p>
<pre class="lang-dart" data-nodeid="17560"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">返回组件信息</span></span>
<span class="hljs-meta">@override</span>
Widget build(BuildContext context) {
  <span class="hljs-keyword">final</span> likeNumModel = Provider.of&lt;LikeNumModel&gt;(context);
  <span class="hljs-keyword">return</span> Container(
    width: <span class="hljs-number">50</span>,
    child: FlatButton(
      padding: EdgeInsets.only(top: <span class="hljs-number">10</span>),
      child: Column(
        children: &lt;Widget&gt;[
          Icon(Icons.thumb_up, color: Colors.grey, size: <span class="hljs-number">36</span>),
          Padding(padding: EdgeInsets.only(top: <span class="hljs-number">2</span>)),
          Text(
            <span class="hljs-string">'<span class="hljs-subst">${likeNumModel.getLikeNum(articleId, likeNum)}</span>'</span>,
            style: TextStyles.commonStyle(),
          ),
        ],
      ),
      onPressed: () =&gt; likeNumModel.like(articleId),
    ),
  );
}
</code></pre>
<p data-nodeid="17561">上述代码是 07 课时已经详细介绍过的部分，其中没有太大的区别，这里需要介绍下 Container 的目的是限制 FlatButton 的大小，避免 FlatButton 产生一些 margin 引起布局问题。以上就完成了单信息流组件的一个设计。</p>
<h3 data-nodeid="17562">总结</h3>
<p data-nodeid="17563">以上就是本课时的所有内容，学完本课时，你要掌握 ListView.separated 的应用，并且了解 ListView 其他构造函数的使用。你要熟练应用 ListView.separated 实现三种内容展示的样式实现方法，并且能进一步熟悉界面效果转化组件设计的实践方法。</p>
<p data-nodeid="17564">本课时已经完成了首页推荐内容，但是还缺乏内容的更新机制，下一课时我将介绍下拉刷新当前数据以及上拉更新列表数据的功能。谢谢。</p>
<p data-nodeid="17565" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="17688">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### *艺：
> ArticleLikeBar会报A RenderFlex overflowed by 1.8 pixels on the right，去掉最外层的row组件就好了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢指出，应该是没有考虑到一些界面元素大小导致的。你可以在 github 上提个 merge request 。

