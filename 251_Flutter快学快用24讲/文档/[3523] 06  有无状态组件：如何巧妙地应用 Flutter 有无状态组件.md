<h3 data-nodeid="14704" class="">无/有状态组件</h3>
<p data-nodeid="14705">由于无状态组件在执行过程中只有一个 build 阶段，在执行期间只会执行一个 build 函数，没有其他生命周期函数，因此在执行速度和效率方面比有状态组件更好。所以在设计组件时，不要任何组件都使用有状态组件进行开发，要根据实际分析情况使用。</p>
<h4 data-nodeid="14706">Flutter 中基础组件介绍</h4>
<p data-nodeid="14707">Flutter 内部包含一些基础的无状态组件，在组件设计的时候，需要对基础组件有一定认识。本课时所使用的 Flutter 基础组件（这里我只简单介绍本课时所使用的组件，更多组件请参考<a href="https://flutterchina.club/widgets/" data-nodeid="14821">官网文档</a>）包括：</p>
<ul data-nodeid="14708">
<li data-nodeid="14709">
<p data-nodeid="14710">Text，文本显示组件，里面包含了文本类相关的样式以及排版相关的配置信息；</p>
</li>
<li data-nodeid="14711">
<p data-nodeid="14712">Image，图片显示组件，里面包含了图片的来源设置，以及图片的配置；</p>
</li>
<li data-nodeid="14713">
<p data-nodeid="14714">Icon，Icon 库，里面是 Flutter 原生支持的一些小的 icon ；</p>
</li>
<li data-nodeid="14715">
<p data-nodeid="14716">FlatButton，包含点击动作的组件；</p>
</li>
<li data-nodeid="14717">
<p data-nodeid="14718">Row，布局组件，在水平方向上依次排列组件；</p>
</li>
<li data-nodeid="14719">
<p data-nodeid="14720">Column，布局组件，在垂直方向上依次排列组件；</p>
</li>
<li data-nodeid="14721">
<p data-nodeid="14722">Container，布局组件，容器组件，这点有点类似于前端中的 body ；</p>
</li>
<li data-nodeid="14723">
<p data-nodeid="14724">Expanded，可以按比例“扩伸” Row、Column 和 Flex 子组件所占用的空间 ，这点就是前端所介绍的 flex 布局设计；</p>
</li>
<li data-nodeid="14725">
<p data-nodeid="14726">Padding，可填充空白区域组件，这点和前端的 padding 功能基本一致；</p>
</li>
<li data-nodeid="14727">
<p data-nodeid="14728">ClipRRect，圆角组件，可以让子组件有圆形边角。</p>
</li>
</ul>
<p data-nodeid="14729">以上组件的具体使用以及参数配置，在<a href="https://flutterchina.club/widgets/" data-nodeid="14836">官网</a>中有具体说明。</p>
<h4 data-nodeid="14730">组件组合要点</h4>
<p data-nodeid="14731">Flutter 功能的开发，可以总结为将基础组件组合并赋予一些交互行为的过程，因此需要掌握组件组合的一些要点。</p>
<p data-nodeid="14732">由于动态组件和静态组件的特点，因此在组合的时候要非常注意，动态组件下的子组件如果过多，则在组件更新的时候，会导致其子组件的全部更新，从而引发性能问题。因此在组件组合的时候需要有一些避免的规则来参考，下面这几点是我自己整理的一套原则。</p>
<ol data-nodeid="14733">
<li data-nodeid="14734">
<p data-nodeid="14735">尽可能减少动态组件下的静态组件；</p>
</li>
<li data-nodeid="14736">
<p data-nodeid="14737">数据来源相同的部分组合为同一组件；</p>
</li>
<li data-nodeid="14738">
<p data-nodeid="14739">使用行或者列作为合并的条件；</p>
</li>
<li data-nodeid="14740">
<p data-nodeid="14741">功能相同的部分，转化为基础组件；</p>
</li>
<li data-nodeid="14742">
<p data-nodeid="14743">合并后根节点的为 Container。</p>
</li>
</ol>
<h3 data-nodeid="14744">组件设计</h3>
<p data-nodeid="14745">当 UI 提供一个交互过来以后，我们要用一套标准的流程来设计组件，减少主观上不合理的设计，接下来主要介绍下我整理的那一套原则。</p>
<p data-nodeid="14746">例如我们现在需要设计一个图 1 的主界面，其次可以进行点赞操作，点赞完成后数字增加 1 。</p>
<p data-nodeid="14747"><img src="https://s0.lgstatic.com/i/image/M00/26/C9/Ciqc1F7zAciAV3kHAAsOqdx9yX8344.png" alt="image (10).png" data-nodeid="14851"><br>
图 1 Two You 的首页推荐界面</p>
<h4 data-nodeid="14748">界面分析</h4>
<p data-nodeid="14749">将设计方案分为以下几个过程：</p>
<ol data-nodeid="14750">
<li data-nodeid="14751">
<p data-nodeid="14752">标记所有界面元素（ Flutter 基础组件），按照数字进行标记，请注意明显相似部分不需要标记；</p>
</li>
<li data-nodeid="14753">
<p data-nodeid="14754">记录相应数字对应的组件名称，并命名展示数据的参数名；</p>
</li>
<li data-nodeid="14755">
<p data-nodeid="14756">标记需要交互和数据变化的数据；</p>
</li>
<li data-nodeid="14757">
<p data-nodeid="14758">将数字和数据合成一个最小组件，并标记组件是有或无状态组件；</p>
</li>
<li data-nodeid="14759">
<p data-nodeid="14760">将第 4 步中的最小组件进行合并，组件组合规则，请参照上面介绍到的“组件组合要点”；</p>
</li>
<li data-nodeid="14761">
<p data-nodeid="14762">完成组件合并后，绘制一张组件图，再次分析动态组件下的静态组件是否合理，如果能拆分尽量拆分。</p>
</li>
</ol>
<p data-nodeid="14763">按照这 6 个步骤我们就可以实现一个界面的组件分析，接下来我们按照图 1 中的界面以及这 6 个步骤实践组件设计。</p>
<h4 data-nodeid="14764">实践设计</h4>
<p data-nodeid="14765">第一步：<strong data-nodeid="14869">标记界面元素</strong>，标记效果如图 2 所示：</p>
<p data-nodeid="14766"><img src="https://s0.lgstatic.com/i/image/M00/26/C9/Ciqc1F7zAdOAel6TAAtwHZn7wQw909.png" alt="image (11).png" data-nodeid="14872"><br>
图 2 组件标记</p>
<p data-nodeid="14767">第二步：记录<strong data-nodeid="14884">组件名称以及组件数据</strong>，以及<strong data-nodeid="14885">标记动态数据</strong>，我们使用下面的表格 1 来处理，并且将每个小组件命名好。</p>
<p data-nodeid="14768"><img src="https://s0.lgstatic.com/i/image/M00/26/C9/Ciqc1F7zAemAA3hwAACRl_hUd40759.png" alt="image (12).png" data-nodeid="14888"><br>
表格 1 组件标记结果</p>
<p data-nodeid="14769">第三步：<strong data-nodeid="14896">组件合并</strong>，主要是将功能一致的组件进行合并，上面的组件中，5 和 6 属于头像信息合并为一个最小组件，7 和 8 为帖子信息合并，9 和 10 为点赞信息合并。</p>
<p data-nodeid="14770" class="">第四部：<strong data-nodeid="14902">创建组件树</strong>，1 由于是单独的一列，因此我们单独一个组件。2 和 3 以及 4 为同一行组件，因此可以合并为一个 Row 组件。然后 2 和 3 垂直排列，组合为一个 Column 组件。5、6、7、8、9、10 都在同一行，因此合并为一个组件。2、3、4、5、6、7、8、9、10 由于存在重复部分，因此合并为一个大的 Row 组件。根据上面的结论，我们绘制组件树如图 3 所示。</p>
<p data-nodeid="16073"><img src="https://s0.lgstatic.com/i/image/M00/31/5D/CgqCHl8MPo6AfA-wAACs_VZfuQw592.png" alt="5.png" data-nodeid="16076"><br>
图 3 组件树</p>




<p data-nodeid="14772">图 3 中的黄色背景的表示动态组件，由于 9 和 10 组成的是最小组件，因此将 9 和 10 作为一个动态组件。所有的叶子节点上都是我们第一步所标记的基础组件，完成后看是否满足我们的设计规则。可以看到动态组件只有一个，在右下角的 Row 中，而该动态组件下只有最小的组件组合，因此是满足我们的设计要求。但为了减少动态组件 10 因状态改变而影响的范围，我们可以将 5、6、7、8 合 并为一个新的组件，将 9 和 10 单独作为一个组件，如图 4 所示。</p>
<p data-nodeid="17717"><img src="https://s0.lgstatic.com/i/image/M00/31/5D/CgqCHl8MPp2AZhKfAADFimfXDyg939.png" alt="6.png" data-nodeid="17720"><br>
图 4 优化后的组件设计图</p>




<p data-nodeid="14774">这样就完成了组件的分析和设计，按照这种方式，我们再实现其中的组件代码，完成界面效果。接下来我们将组件进行命名，将表格 1 根据上图的设计重新修改下，如表格 2 。</p>
<p data-nodeid="14775"><img src="https://s0.lgstatic.com/i/image/M00/26/C9/Ciqc1F7zAgaAJ59xAABi3IF6Ycw572.png" alt="image (15).png" data-nodeid="14917"><br>
表格 2 最终组件设计结构</p>
<p data-nodeid="14776">接下来我们就使用代码去实现这部分功能。</p>
<h3 data-nodeid="14777">综合实践</h3>
<p data-nodeid="14778">根据表格 2 的设计结构，我们先将目录层级结构，以及相应目录下的文件创建好。因为在表格中，组件都已经命名好了，所以创建就比较简单，如图 5 所示的目录结构。图 5 中的 struct 用来做数据结构描述，类似 TypeScript 中的 interface 作用，避免因为数据结构问题导致的异常。</p>
<p data-nodeid="14779"><img src="https://s0.lgstatic.com/i/image/M00/26/C9/Ciqc1F7zAg2AAeVzAACV_ulkeyM179.png" alt="image (16).png" data-nodeid="14925"><br>
图 5 组件目录结构</p>
<p data-nodeid="14780">接下来我先从底层组件依次实现，其次将组件的数据都作为参数传递进来即可。下面的代码，我只介绍核心部分，其他代码大家可以参考 Github 中的 06 课时源码。</p>
<h4 data-nodeid="14781">基础组件</h4>
<p data-nodeid="14782">下面我们主要看几个核心的组件实现。</p>
<p data-nodeid="14783"><strong data-nodeid="14938">article_like_bar</strong></p>
<p data-nodeid="14784">该组件包括点赞数动态变量的声明以及点赞行为（点赞行为中触发的逻辑是，增加点赞数 state 的值）两部分内容，具体代码实现如下：</p>
<pre class="lang-dart" data-nodeid="14785"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">帖子文章的赞组件</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 包括点赞组件 icon ，以及组件点击效果</span></span>
<span class="hljs-comment">/// <span class="markdown">需要外部参数[likeNum],点赞数量</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ArticleLikeBar</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatefulWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">外部传入参数</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">int</span> likeNum;
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  <span class="hljs-keyword">const</span> ArticleLikeBar({Key key, <span class="hljs-keyword">this</span>.likeNum}) : <span class="hljs-keyword">super</span>(key: key);
  <span class="hljs-meta">@override</span>
  createState() =&gt; ArticleLikeBarState();
}
<span class="hljs-comment">/// <span class="markdown">帖子d文章的赞组件的状态管理类</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 内部包括组件的点赞效果，点击后触发数字更新操作</span></span>
<span class="hljs-comment">/// <span class="markdown">[likeNum] 为状态组件可变数据</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ArticleLikeBarState</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">State</span>&lt;<span class="hljs-title">ArticleLikeBar</span>&gt; </span>{
  <span class="hljs-comment">/// <span class="markdown">状态 state</span></span>
  <span class="hljs-built_in">int</span> likeNum;
  <span class="hljs-meta">@override</span>
  <span class="hljs-keyword">void</span> initState() {
    <span class="hljs-keyword">super</span>.initState();
    likeNum ??= widget.likeNum;
  }
  <span class="hljs-comment">/// <span class="markdown">点赞动作效果，点击后[likeNum]加1</span></span>
  <span class="hljs-keyword">void</span> like() {
    setState(() {
      likeNum = ++likeNum;
    });
  }
  <span class="hljs-comment">/// <span class="markdown">有状态类返回组件信息</span></span>
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> Row(
      children: &lt;Widget&gt;[
        Icon(Icons.thumb_up, color: Colors.grey, size: <span class="hljs-number">18</span>),
        Padding(padding: EdgeInsets.only(left: <span class="hljs-number">10</span>)),
        FlatButton(
          child: Text(<span class="hljs-string">'<span class="hljs-subst">$likeNum</span>'</span>),
          onPressed: () =&gt; like(),
        ),
      ],
    );
  }
}
</code></pre>
<p data-nodeid="14786">学到本课时，除了 build 中的部分代码比较陌生，其他部分的代码应该是非常熟悉的，因此这里简单介绍了 build 中的逻辑。build 中使用了 Row 布局组件，然后包含了三个最基础组件：Icon、Padding 和 FlatButton ，而 FlatButton 中又包含了基础组件 Text ，总结下 build 主要是完成界面组件组合以及增加组件交互行为这两方面的逻辑。</p>
<h4 data-nodeid="14787">二级组件</h4>
<p data-nodeid="14788">上面介绍了一个一级组件的代码，我们再来看一个二级组件 article_card 的代码实现，在表格 2 中，可以看到其需要引入三个一级组件，因此在头部需要引入代码如下：</p>
<pre class="lang-dart" data-nodeid="14789"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/util/struct/article_summary_struct.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/util/struct/user_info_struct.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/home_page/article_bottom_bar.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/home_page/article_like_bar.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/home_page/article_summary.dart'</span>;
</code></pre>
<p data-nodeid="14790">第 1 行是 Flutter 基础组件，第 3 和 4 行是我们上面介绍的数据结构描述类库，第 6、7 和 8 行 则是三个一级组件的库。</p>
<p data-nodeid="14791">因为 article_card 是一个静态组件，所以只需要继承 StatelessWidget 。接下来实现 build 方法，由于子组件需要用户信息和帖子信息，所以该组件包含两个参数。</p>
<pre class="lang-dart" data-nodeid="14792"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">此为帖子描述类，包括了帖子UI中的所有元素</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ArticleCard</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">传入的用户信息</span></span>
  <span class="hljs-keyword">final</span> UserInfoStruct userInfo;
  <span class="hljs-comment">/// <span class="markdown">传入的帖子信息</span></span>
  <span class="hljs-keyword">final</span> ArticleSummaryStruct articleInfo;
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  <span class="hljs-keyword">const</span> ArticleCard({Key key, <span class="hljs-keyword">this</span>.userInfo, <span class="hljs-keyword">this</span>.articleInfo})
      : <span class="hljs-keyword">super</span>(key: key);
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
  }
}
</code></pre>
<p data-nodeid="14793">上面代码中的第 3 到 10 行是定义无状态类的参数以及构造函数，接下来我们看看 build 函数的实现，代码如下：</p>
<pre class="lang-dart" data-nodeid="14794"><code data-language="dart"><span class="hljs-meta">@override</span>
Widget build(BuildContext context) {
  <span class="hljs-keyword">return</span> Column(
    children: &lt;Widget&gt;[
      ArticleSummary(
          title: articleInfo.title,
          summary: articleInfo.summary,
          articleImage: articleInfo.articleImage),
      Row(
        children: &lt;Widget&gt;[
          ArticleBottomBar(
              nickname: userInfo.nickname,
              headerImage: userInfo.headerImage,
              commentNum: articleInfo.commentNum),
          ArticleLikeBar(likeNum: articleInfo.likeNum),
        ],
      ),
    ],
  );
}
</code></pre>
<p data-nodeid="14795">UI 稿中包含两部分内容展示，第一部分是帖子的概要信息，第二部分是帖子的相关作者以及点赞评论信息，因此可以使用垂直组件 Column ，竖着排列这两部分内容。由于在状态栏这一行中又包含有作者信息和评论点赞信息，因此使用 Row 组件包裹两个一级组件 ArticleBottomBar 和 ArticleLikeBar 。这样就完成了该组件无状态类组件的设计，我们看下整个代码实现。</p>
<pre class="lang-dart" data-nodeid="14796"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/util/struct/article_summary_struct.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/util/struct/user_info_struct.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/home_page/article_bottom_bar.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/home_page/article_like_bar.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/home_page/article_summary.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">此为帖子描述类，包括了帖子UI中的所有元素</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ArticleCard</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">传入的用户信息</span></span>
  <span class="hljs-keyword">final</span> UserInfoStruct userInfo;
  <span class="hljs-comment">/// <span class="markdown">传入的帖子信息</span></span>
  <span class="hljs-keyword">final</span> ArticleSummaryStruct articleInfo;
  <span class="hljs-comment">/// <span class="markdown">构造函数</span></span>
  <span class="hljs-keyword">const</span> ArticleCard({Key key, <span class="hljs-keyword">this</span>.userInfo, <span class="hljs-keyword">this</span>.articleInfo})
      : <span class="hljs-keyword">super</span>(key: key);
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> Column(
      children: &lt;Widget&gt;[
        ArticleSummary(
            title: articleInfo.title,
            summary: articleInfo.summary,
            articleImage: articleInfo.articleImage),
        Row(
          children: &lt;Widget&gt;[
            ArticleBottomBar(
                nickname: userInfo.nickname,
                headerImage: userInfo.headerImage,
                commentNum: articleInfo.commentNum),
            ArticleLikeBar(likeNum: articleInfo.likeNum),
          ],
        ),
      ],
    );
  }
}
</code></pre>
<h4 data-nodeid="14797">页面组件</h4>
<p data-nodeid="14798">其他组件的开发方式按照上面一个无状态组件和一个有状态组件的方式开发即可，最后再看下页面组件 home_page.dart ，因为这是一个静态组件页面，所以相对较为简单，代码如下：</p>
<pre class="lang-dart" data-nodeid="14799"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:flutter/material.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/util/struct/article_summary_struct.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/util/struct/user_info_struct.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/common/banner_info.dart'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you/widgets/home_page/article_card.dart'</span>;
<span class="hljs-comment">/// <span class="markdown">首页列表信息</span></span>
<span class="hljs-comment">///
<span class="markdown">/// 展示banner和帖子信息</span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HomePage</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-comment">/// <span class="markdown">banner 地址信息</span></span>
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">String</span> bannerImage =
      <span class="hljs-string">'https://img.089t.com/content/20200227/osbbw9upeelfqnxnwt0glcht.jpg'</span>;
  <span class="hljs-comment">/// <span class="markdown">帖子标题</span></span>
  <span class="hljs-keyword">final</span> UserInfoStruct userInfo = UserInfoStruct(<span class="hljs-string">'flutter'</span>,
      <span class="hljs-string">'https://i.pinimg.com/originals/1f/00/27/1f0027a3a80f470bcfa5de596507f9f4.png'</span>);
  <span class="hljs-comment">/// <span class="markdown">帖子概要描述信息</span></span>
  <span class="hljs-keyword">final</span> ArticleSummaryStruct articleInfo = ArticleSummaryStruct(
      <span class="hljs-string">'你好，交个朋友'</span>,
      <span class="hljs-string">'我是一个小可爱'</span>,
      <span class="hljs-string">'https://i.pinimg.com/originals/e0/64/4b/e0644bd2f13db50d0ef6a4df5a756fd9.png'</span>,
      <span class="hljs-number">20</span>,
      <span class="hljs-number">30</span>);
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> Container(
      child: Column(
        children: &lt;Widget&gt;[
          BannerInfo(bannerImage: bannerImage),
          ArticleCard(
            userInfo: userInfo,
            articleInfo: articleInfo,
          ),
        ],
      ),
    );
  }
}
</code></pre>
<ul data-nodeid="14800">
<li data-nodeid="14801">
<p data-nodeid="14802">import 库，import 库包括 Flutter 基础库，两个数据结构类定义库，以及 banner 组件和 articile_card 库；</p>
</li>
<li data-nodeid="14803">
<p data-nodeid="14804">初始化几个本次测试需要的数据，banner 数据、userInfo 数据和 article 内容数据；</p>
</li>
<li data-nodeid="14805">
<p data-nodeid="14806">build 返回组件内容，使用 Column 垂直排列 banner 组件和 article 组件。</p>
</li>
</ul>
<p data-nodeid="14807">以上就实现了该组件的所有功能点，首先还是运行代码检查工具，使用下面的运行命令。</p>
<pre class="lang-plain" data-nodeid="14808"><code data-language="plain">sh format_check.sh
</code></pre>
<p data-nodeid="14809">确认代码无误后，再打开手机模拟器，运行程序，即可看到图 6 的界面效果。</p>
<p data-nodeid="14810"><img src="https://s0.lgstatic.com/i/image/M00/26/CA/Ciqc1F7zAi6AIamAAAnwMrGReX8255.png" alt="image (17).png" data-nodeid="14964"><br>
图 6 应用运行效果</p>
<h3 data-nodeid="14811">总结</h3>
<p data-nodeid="14812">本课时介绍了有无状态组件的一些特点以及在组件设计选择的时候需要注意的规则，接下来实践了组件设计的方法。学完本课时你应该掌握界面的组件设计思想，并根据组件设计实现具体的交互界面。</p>
<p data-nodeid="14813">以上就是本课时的所有内容，下一课时我将介绍 Flutter 中有状态组件的状态管理，学完本课时以及下一课时，你将可以掌握复杂的交互界面开发技巧。</p>
<p data-nodeid="14814" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="14972">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### *鹏：
> 您好老师，最后组合成HomePage的时候，是把Column包在Container里面返回的，想问下如果直接returnColumn这个组件有影响吗，因为看很多代码示例都有包含在Container里面的，Container这个组件具体的作用大吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 直接返回时没有影响的，但是有 Container 容器包裹后，后续你想做一下布局调整时比较简单的，因为容器 Container 包含了比较多而复杂的布局设计。建议是最好有容器包裹着，当然也是可以不需要的。

##### **锋：
> 对UI进行剥离分组，再逐个分析组合组件，确实很实用，感谢大神！

##### Lemon：
> 一行行的敲完代码，收获不少，初学dart 语法。

##### **威：
> const ArticleLikeBar({Key key, this.likeNum}) : super(key: key){Key key,this.likeNum}是一个sets吗？super(key: key)中key: key是什么意思？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里的 key 主要目的是让组件有一个唯一标识，例如你有一个列表需要显示，但是如果没有唯一key区分的话，就相当于组件之间没有区分，那么在组件操作时，比如删除某个组件或者新增修改某个组件，都会导致整个列表组件的重新渲染，从而影响性能。这个是key的主要作用，这个key也是组件的一个可选参数。

##### *斌：
> const ArticleLikeBar({Key key, this.likeNum}) : super(key: key);老师这里的Key 是什么含义呢？还有super

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; key的意思是给组件的唯一标识，其实就是标识为组件唯一识别的参数。这里一般是一个可选参数，key在flutter中是一个优化项，和前端的设置组件的key是相似的，避免在组件移动、删除或者新增时产生的性能问题。super就是关键词，通过super可以调用父类的方法。

##### *鑫：
> 老师您好，图3和图片4的改动没能理解.这里动态更新点赞数量，只会更新当前组件.至于其他组件并不影响吧.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 问题提的很好，确实这里会有点误解，我再说明下。图 3 我描述的是将 5 6 7 8 9 10 合并为一个组件，而图 4 是将 5 6 7 8 作为一个组件，9 10 作为一个组件。主要是这点差异，所以这里就是为了避免 10 的动态性而影响到 5 6 7 8。这里再补充一下，在规则里面我们说了合并功能相同组件和数据来源相同作为一个组件的目的都是为了减少组件，所以会将 5 6 7 8合并一个组件。

##### *政：
> 关于第一条尽量减少动态组件下的静态组件。如果刷新整个页面，动态组件内部组件多少没什么区别影响，因为这些静态组件都是要刷新的。如果考虑动态组件内部刷新存在组件构建优势。这样理解可以吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的哈，如果整体刷新是没有优势。单针对动态组件，如果设计不合理，都堆积在一个动态组件下，就会导致动态组件一刷新，全部子组件都更新了。但是很多情况下，很多子组件没有依赖某些状态，是没有必要和其他动态组件合入到一个组件中。

##### **彬：
> 因此合并为一个组件。2、3、4、5、6、7、8、9、10 由于存在重复部分，因此合并为一个大的 Row 组件这里 不应该是一个column组件吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里其实也没有太大关系，使用Row和Column。不过根据后续如果需要多列的话，的确使用Column来显示较好。在后续章节中，我们还是使用ListView来作为列的控件。具体往后这部分还会介绍到。

##### **宇：
> 2,3,4组成的Row组件 和 5,6,7,8,9,10组成的Row组件，应该放在一个Column组件里面吧？课件里显示的还是放在一个Row组件里。<br>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是Row里面哈，因为这几个组件是并列一排显示。

##### **亮：
> 上个项目基本全用了stf，经过这节课又学到了😂

