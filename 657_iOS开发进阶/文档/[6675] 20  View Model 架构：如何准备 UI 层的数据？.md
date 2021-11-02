<p data-nodeid="1552" class="">UI 是 App 的重要组成部分，因为所有 App 都必须呈现 UI，并接收用户的事件。为了让 UI 能正确显示，我们需要把 Model 数据进行转换。例如，当我们显示图片的时候，需要把字符串类型的 URL 转换成 iOS 所支持 URL 类型；当显示时间信息时，需要把 UTC 时间值转换成设备所在的时区。</p>
<p data-nodeid="1553">不过存在一个问题，如果我们把所有类型转换的逻辑都放在 UI/View 层里面，作为 View 层的 View Controller 往往会变得越来越臃肿。 为了避免这一情况，我使用了 MVVM 模式和 RxSwift 来架构 Moments App。MVVM 模式的核心部分是 ViewModel 模块，主要用于把 Model 转换成 UI/View 层所需的数据。为了简化转换的工作，我使用了 RxSwift 的操作符（Operator）。</p>
<p data-nodeid="1554">所以，在这一讲中，我会和你介绍下 ViewModel 模式是怎样工作的，以及如何使用 RxSwift 里常用的操作符。</p>
<h3 data-nodeid="1555">ViewModel 模式的架构</h3>
<p data-nodeid="1556">首先我们以朋友圈功能为例，看看 ViewModel 模式的架构图。</p>
<p data-nodeid="1557"><img src="https://s0.lgstatic.com/i/image6/M00/3C/0F/CioPOWCH7COAXcMTAAPT7Gr7yvg197.png" alt="Drawing 1.png" data-nodeid="1669"></p>
<p data-nodeid="1558"><strong data-nodeid="1676">View 模块</strong>负责呈现 UI，并接收用户的事件。在朋友圈功能中，<code data-backticks="1" data-nodeid="1674">MomentsTimelineViewController</code>负责呈现朋友圈的时间轴列表。为了正确显示该页面，我们需要为它准备好一些的数据，例如朋友的名字，朋友头像的 URL 等等，那些数据可以从 ViewModel 模块中读取。</p>
<p data-nodeid="1559"><strong data-nodeid="1691">ViewModel 模块</strong>是 MVVM 模式的核心，该模块由两个重要的协议所组成：<code data-backticks="1" data-nodeid="1681">ListViewModel</code>和<code data-backticks="1" data-nodeid="1683">ListItemViewModel</code>。其中<code data-backticks="1" data-nodeid="1685">ListViewModel</code>协议用于定义列表页面所需的 ViewModel，而<code data-backticks="1" data-nodeid="1687">ListItemViewModel</code>用于定义每一条列表项所需的 ViewModel。当他们需要读写数据时，会调用 Repository 模块。比如在朋友圈功能里面，它们都调用<code data-backticks="1" data-nodeid="1689">MoomentsRepoType</code>来读写数据。</p>
<h3 data-nodeid="1560">ViewModel 模式的实现</h3>
<p data-nodeid="1561">有了上述的架构图，我们就可以看看 ViewModel 模块是怎样实现的。首先看一下<code data-backticks="1" data-nodeid="1694">ListViewModel</code>协议的定义。</p>
<pre class="lang-swift" data-nodeid="1562"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">protocol</span> <span class="hljs-title">ListViewModel</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">var</span> hasContent: <span class="hljs-type">Observable</span>&lt;<span class="hljs-type">Bool</span>&gt; { <span class="hljs-keyword">get</span> }
&nbsp; &nbsp; <span class="hljs-keyword">var</span> hasError: <span class="hljs-type">BehaviorSubject</span>&lt;<span class="hljs-type">Bool</span>&gt; { <span class="hljs-keyword">get</span> }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">trackScreenviews</span><span class="hljs-params">()</span></span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">loadItems</span><span class="hljs-params">()</span></span> -&gt; <span class="hljs-type">Observable</span>&lt;<span class="hljs-type">Void</span>&gt;
    <span class="hljs-keyword">var</span> listItems: <span class="hljs-type">BehaviorSubject</span>&lt;[<span class="hljs-type">SectionModel</span>&lt;<span class="hljs-type">String</span>, <span class="hljs-type">ListItemViewModel</span>&gt;]&gt; { <span class="hljs-keyword">get</span> }
}
</code></pre>
<p data-nodeid="1563">下面我们逐一介绍该协议的各个属性与方法。<code data-backticks="1" data-nodeid="1697">hasContent</code>属性用于通知 UI 是否有内容。例如，当 BFF 没有返回数据时，我们可以在页面上提示用户“目前还没有朋友圈信息，可以添加好友来查看更多的朋友圈信息”。</p>
<p data-nodeid="1564">为了代码共享，我们为<code data-backticks="1" data-nodeid="1700">hasContent</code>属性提供了一个默认的实现，代码如下。</p>
<pre class="lang-swift" data-nodeid="1565"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">extension</span> <span class="hljs-title">ListViewModel</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">var</span> hasContent: <span class="hljs-type">Observable</span>&lt;<span class="hljs-type">Bool</span>&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> listItems
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .<span class="hljs-built_in">map</span>(\.isEmpty)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .distinctUntilChanged()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .asObservable()
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1566">这个方法使用<code data-backticks="1" data-nodeid="1703">map</code>和<code data-backticks="1" data-nodeid="1705">distinctUntilChanged</code>操作符来把<code data-backticks="1" data-nodeid="1707">listItems</code>转换成 Bool 类型的<code data-backticks="1" data-nodeid="1709">hasContent</code>。其中<code data-backticks="1" data-nodeid="1711">map</code>用于提取<code data-backticks="1" data-nodeid="1713">listItems</code>里的数组并检查是否为空，<code data-backticks="1" data-nodeid="1715">distinctUntilChanged</code>用来保证只有在值发生改变时才发送新事件。</p>
<p data-nodeid="1567"><code data-backticks="1" data-nodeid="1717">hasError</code>属性是一个<code data-backticks="1" data-nodeid="1719">BehaviorSubject</code>，其初始值为<code data-backticks="1" data-nodeid="1721">false</code>。它用于通知 UI 是否需要显示错误信息。</p>
<p data-nodeid="1568"><code data-backticks="1" data-nodeid="1723">trackScreenviews()</code>方法用于发送用户行为数据。而<code data-backticks="1" data-nodeid="1725">loadItems() -&gt; Observable&lt;Void&gt;</code>方法用于读取数据。</p>
<p data-nodeid="1569">最后看一下<code data-backticks="1" data-nodeid="1728">listItems</code>属性。 该属性用于准备 TableView 所需的数据，其存放了类型为<code data-backticks="1" data-nodeid="1730">ListItemViewModel</code>的数据。<code data-backticks="1" data-nodeid="1732">ListItemViewModel</code>能为 TableView 的各个 Cell 提供所需数据。该协议只定义一个名为<code data-backticks="1" data-nodeid="1734">reuseIdentifier</code>的静态属性 ，如下所示。</p>
<pre class="lang-swift" data-nodeid="1570"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">protocol</span> <span class="hljs-title">ListItemViewModel</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">static</span> <span class="hljs-keyword">var</span> reuseIdentifier: <span class="hljs-type">String</span> { <span class="hljs-keyword">get</span> }
}
<span class="hljs-class"><span class="hljs-keyword">extension</span> <span class="hljs-title">ListItemViewModel</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">static</span> <span class="hljs-keyword">var</span> reuseIdentifier: <span class="hljs-type">String</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-type">String</span>(describing: <span class="hljs-keyword">self</span>)
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1571"><code data-backticks="1" data-nodeid="1736">reuseIdentifier</code>属性作为 TableView Cell 的唯一标示，为了重用，我们通过协议扩展来为该属性提供一个默认的实现并把类型的名字作为字符串进行返回。<br>
上述就是<code data-backticks="1" data-nodeid="1740">ListViewModel</code>协议的定义，接下来看它的实现结构体<code data-backticks="1" data-nodeid="1742">MomentsTimelineViewModel</code>。</p>
<p data-nodeid="1572">由于<code data-backticks="1" data-nodeid="1745">MomentsTimelineViewModel</code>遵循了<code data-backticks="1" data-nodeid="1747">ListViewModel</code>协议，因此需要实现了该协议中<code data-backticks="1" data-nodeid="1749">listItems</code>和<code data-backticks="1" data-nodeid="1751">hasError</code>属性以及<code data-backticks="1" data-nodeid="1753">loadItems()</code>和<code data-backticks="1" data-nodeid="1755">trackScreenviews()</code>方法。我们首先看一下<code data-backticks="1" data-nodeid="1757">loadItems()</code>方法的实现。</p>
<pre class="lang-swift" data-nodeid="1573"><code data-language="swift"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">loadItems</span><span class="hljs-params">()</span></span> -&gt; <span class="hljs-type">Observable</span>&lt;<span class="hljs-type">Void</span>&gt; {
    <span class="hljs-keyword">return</span> momentsRepo.getMoments(userID: userID)
}
</code></pre>
<p data-nodeid="1574">当 ViewModel 需要读取数据的时候，会调用 Repository 模块的组件，在朋友圈功能中，我们调用了<code data-backticks="1" data-nodeid="1760">MomentsRepoType</code>的<code data-backticks="1" data-nodeid="1762">getMoments()</code>方法来读取数据。</p>
<p data-nodeid="1575">接着看看<code data-backticks="1" data-nodeid="1765">trackScreenviews()</code>方法的实现。在该方法里面，我们调用了<code data-backticks="1" data-nodeid="1767">TrackingRepoType</code>的<code data-backticks="1" data-nodeid="1769">trackScreenviews()</code>方法来发送用户的行为数据，具体实现如下。</p>
<pre class="lang-swift" data-nodeid="1576"><code data-language="swift"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">trackScreenviews</span><span class="hljs-params">()</span></span> {
    trackingRepo.trackScreenviews(<span class="hljs-type">ScreenviewsTrackingEvent</span>(screenName: <span class="hljs-type">L10n</span>.<span class="hljs-type">Tracking</span>.momentsScreen, screenClass: <span class="hljs-type">String</span>(describing: <span class="hljs-keyword">self</span>)))
 }
</code></pre>
<p data-nodeid="1577"><strong data-nodeid="1775">ViewModel 模块的一个核心功能，是把 Model 数据转换为用于 UI 呈现所需的 ViewModel 数据</strong>，我通过下面代码看它是怎样转换的。</p>
<pre class="lang-swift" data-nodeid="1578"><code data-language="swift"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">setupBindings</span><span class="hljs-params">()</span></span> {
 momentsRepo.momentsDetails
     .<span class="hljs-built_in">map</span> {
         [<span class="hljs-type">UserProfileListItemViewModel</span>(userDetails: $<span class="hljs-number">0</span>.userDetails)]
             + $<span class="hljs-number">0</span>.moments.<span class="hljs-built_in">map</span> { <span class="hljs-type">MomentListItemViewModel</span>(moment: $<span class="hljs-number">0</span>) }
     }
     .subscribe(onNext: {
         listItems.onNext([<span class="hljs-type">SectionModel</span>(model: <span class="hljs-string">""</span>, items: $<span class="hljs-number">0</span>)])
     }, onError: { <span class="hljs-number">_</span> <span class="hljs-keyword">in</span>
         hasError.onNext(<span class="hljs-literal">true</span>)
     })
     .disposed(by: disposeBag)
}
</code></pre>
<p data-nodeid="1579">从代码中你可以发现，我们订阅了<code data-backticks="1" data-nodeid="1777">momentsRepo</code>的<code data-backticks="1" data-nodeid="1779">momentsDetails</code>属性，接收来自 Model 的数据更新。因为该属性的类型是<code data-backticks="1" data-nodeid="1781">MomentsDetails</code>，而 View 层用所需的数据类型为<code data-backticks="1" data-nodeid="1783">ListItemViewModel</code>。我们通过 map 操作符来进行类型转换，在转换成功后，调用<code data-backticks="1" data-nodeid="1785">listItems</code>的<code data-backticks="1" data-nodeid="1787">onNext()</code>方法把准备好的 ViewModel 数据发送给 UI。如果发生错误，就通过<code data-backticks="1" data-nodeid="1789">hasError</code>属性发送出错信息。</p>
<p data-nodeid="1580">在 map 操作符的转换过程中，我们分别使用了<code data-backticks="1" data-nodeid="1792">UserProfileListItemViewModel</code>和<code data-backticks="1" data-nodeid="1794">MomentListItemViewModel</code>结构体来转换用户简介信息和朋友圈条目信息。这两个结构体都遵循了<code data-backticks="1" data-nodeid="1796">ListItemViewModel</code>协议。</p>
<p data-nodeid="1581">接下来是它们的实现，首先看一下<code data-backticks="1" data-nodeid="1799">UserProfileListItemViewModel</code>。</p>
<pre class="lang-swift" data-nodeid="1582"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">UserProfileListItemViewModel</span>: <span class="hljs-title">ListItemViewModel</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">let</span> name: <span class="hljs-type">String</span>
&nbsp; &nbsp; <span class="hljs-keyword">let</span> avatarURL: <span class="hljs-type">URL?</span>
&nbsp; &nbsp; <span class="hljs-keyword">let</span> backgroundImageURL: <span class="hljs-type">URL?</span>
&nbsp; &nbsp; <span class="hljs-keyword">init</span>(userDetails: <span class="hljs-type">MomentsDetails</span>.<span class="hljs-type">UserDetails</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; name = userDetails.name
&nbsp; &nbsp; &nbsp; &nbsp; avatarURL = <span class="hljs-type">URL</span>(string: userDetails.avatar)
&nbsp; &nbsp; &nbsp; &nbsp; backgroundImageURL = <span class="hljs-type">URL</span>(string: userDetails.backgroundImage)
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1583">该结构体只包含了三个属性：<code data-backticks="1" data-nodeid="1802">name</code>、<code data-backticks="1" data-nodeid="1804">avatarURL</code>和<code data-backticks="1" data-nodeid="1806">backgroundImageURL</code>。</p>
<p data-nodeid="1584">其中，由于<code data-backticks="1" data-nodeid="1809">name</code>属性的类型与<code data-backticks="1" data-nodeid="1811">MomentsDetails.UserDetails</code>中<code data-backticks="1" data-nodeid="1813">name</code>属性的类型都是字符串，我们只需要直接赋值就可以了。</p>
<p data-nodeid="1585">而<code data-backticks="1" data-nodeid="1816">avatarURL</code>和<code data-backticks="1" data-nodeid="1818">backgroundImageURL</code>用于在 UI 上显示图片。因为 BFF 返回的 URL 值都是字符串类型，我们需要把字符串转换成<code data-backticks="1" data-nodeid="1820">URL</code>类型。所有的转换工作我都放在<code data-backticks="1" data-nodeid="1822">init(userDetails: MomentsDetails.UserDetails)</code>方法里面完成，我们只需要调用<code data-backticks="1" data-nodeid="1824">URL</code>的初始化函数即可。</p>
<p data-nodeid="1586">接着看一下<code data-backticks="1" data-nodeid="1827">MomentListItemViewModel</code>结构体，它也是负责把 Model 的数据类型转换成用于 View 层显示 UI 的 ViewModel 数据。其转换的逻辑也封装在<code data-backticks="1" data-nodeid="1829">init()</code>方法中，我们一起看看该方法是如何工作的。</p>
<pre class="lang-swift" data-nodeid="1587"><code data-language="swift"><span class="hljs-keyword">init</span>(moment: <span class="hljs-type">MomentsDetails</span>.<span class="hljs-type">Moment</span>, now: <span class="hljs-type">Date</span> = <span class="hljs-type">Date</span>(), relativeDateTimeFormatter: <span class="hljs-type">RelativeDateTimeFormatterType</span> = <span class="hljs-type">RelativeDateTimeFormatter</span>()) {
&nbsp; &nbsp; userAvatarURL = <span class="hljs-type">URL</span>(string: moment.userDetails.avatar)
&nbsp; &nbsp; userName = moment.userDetails.name
&nbsp; &nbsp; title = moment.title

&nbsp; &nbsp; <span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> firstPhoto = moment.photos.first {
&nbsp; &nbsp; &nbsp; &nbsp; photoURL = <span class="hljs-type">URL</span>(string: firstPhoto)
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; photoURL = <span class="hljs-literal">nil</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">var</span> formatter = relativeDateTimeFormatter
&nbsp; &nbsp; formatter.unitsStyle = .full
&nbsp; &nbsp; <span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> timeInterval = <span class="hljs-type">TimeInterval</span>(moment.createdDate) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">let</span> createdDate = <span class="hljs-type">Date</span>(timeIntervalSince1970: timeInterval)
&nbsp; &nbsp; &nbsp; &nbsp; postDateDescription = formatter.localizedString(<span class="hljs-keyword">for</span>: createdDate, relativeTo: now)
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; postDateDescription = <span class="hljs-literal">nil</span>
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1588"><code data-backticks="1" data-nodeid="1831">userName</code>和<code data-backticks="1" data-nodeid="1833">title</code>属性都是字符串类型，只需要简单的赋值就可以了。而<code data-backticks="1" data-nodeid="1835">userAvatarURL</code>和<code data-backticks="1" data-nodeid="1837">photoURL</code>属性需要把字符串转换为<code data-backticks="1" data-nodeid="1839">URL</code>类型来呈现图片。</p>
<p data-nodeid="1589"><code data-backticks="1" data-nodeid="1841">postDateDescription</code>属性相对复杂些，它的用途是显示一个相对的时间值，例如 “5 分钟前”“2 小时前”等。我们需要把朋友圈信息生成的时间与当前时间进行对比，然后根据手机上的语言配置来显示相对时间值。</p>
<h3 data-nodeid="1590">RxSwift 操作符</h3>
<p data-nodeid="1591">ViewModel 的核心功能是把 Model 数据转换为用于 UI 呈现所需的数据。其实<strong data-nodeid="1849">RxSwift 的操作符就是负责转换的，使用合适的操作符能帮我们减少代码量并提高生产力</strong>。因此我建议你把 RxSwift 所提供的所有操作符都看一遍，然后在实际工作再挑选合适的来满足业务需求。</p>
<p data-nodeid="1592">在这里，我着重介绍下过<strong data-nodeid="1855">滤操作符，转换操作符和合并操作符</strong>中常用的 filter、distinctUntilChanged、map 和 combineLatest 等用法。</p>
<h4 data-nodeid="1593">过滤操作符</h4>
<p data-nodeid="1594">过滤操作符用于过滤事件，我们可以使用过滤操作符把订阅者不关心的事件给过滤掉。常用的过滤操作符有 filter 和 distinctUntilChanged。</p>
<p data-nodeid="1595"><strong data-nodeid="1862">filter</strong>操作符常用于通过规则过滤不需要的事件，例如在朋友圈功能里面，可以把发布时间早于一天前的信息过滤掉不显示。为了方便理解，我就以几个数字来解释下。如下所示，有 2、23、5、60、1、31，我想把小于 10 的数过滤掉，就可以通过 filter 设置过滤规则，然后打印出来的数字就是 23、 60、31。代码示例如下。</p>
<pre class="lang-swift" data-nodeid="1596"><code data-language="swift"><span class="hljs-type">Observable</span>.of(<span class="hljs-number">2</span>, <span class="hljs-number">23</span>, <span class="hljs-number">5</span>, <span class="hljs-number">60</span>, <span class="hljs-number">1</span>, <span class="hljs-number">31</span>)
  &nbsp; .<span class="hljs-built_in">filter</span> { $<span class="hljs-number">0</span> &gt; <span class="hljs-number">10</span> }
  &nbsp; .subscribe(onNext: {
  &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
  &nbsp; })
  &nbsp; .disposed(by: disposeBag)
</code></pre>
<p data-nodeid="1597"><img src="https://s0.lgstatic.com/i/image6/M01/3C/06/Cgp9HWCH7E6AcOAxAAD0m4P1ZAs382.png" alt="Drawing 3.png" data-nodeid="1865"></p>
<div data-nodeid="1598"><p style="text-align:center">过滤操作符 filter 的效果</p></div>
<p data-nodeid="1599"><strong data-nodeid="1870">distinctUntilChanged</strong>用于把相同的事件过滤掉。如下面例子中的第二个 1 和第四个 2，使用distinctUntilChanged 就可以把它们给过滤掉，然后打印出 1、 2、 1。代码和图例如下所示。</p>
<pre class="lang-swift" data-nodeid="1600"><code data-language="swift"><span class="hljs-type">Observable</span>.of(<span class="hljs-number">1</span>, <span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">2</span>, <span class="hljs-number">1</span>)
    .distinctUntilChanged()
    .subscribe(onNext: {
        <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
    })
    .disposed(by: disposeBag)
</code></pre>
<p data-nodeid="1601"><img src="https://s0.lgstatic.com/i/image6/M01/3C/06/Cgp9HWCH7Q-AVcVjAAEFCx3nsK4458.png" alt="Drawing 5.png" data-nodeid="1873"></p>
<div data-nodeid="1602"><p style="text-align:center">过滤操作符 distinctUntilChanged 的效果</p></div>
<p data-nodeid="1603">除了相同的事件，我们还可以使用操作符<strong data-nodeid="1879">distinctUntilChanged</strong>过滤掉相同的状态，从而避免频繁更新 UI。例如，我们先使用本地缓存数据呈现 UI，然后发起网络请求。当请求成功以后可以把结果数据与缓存进行对比，如果数据一致就没必要再次更新 UI。</p>
<h4 data-nodeid="1604">转换操作符</h4>
<p data-nodeid="1605">转换操作符非常实用，能帮助我们从一种数据类型转变成另外一种类型，例如我们可以把用于数据传输和存储的 Model 类型转换成用于 UI 呈现的 ViewModel 类型。在这里，我就以几个常用的转换操作符 map，compactMap 和 flapMap 来介绍下如何使用它们。</p>
<p data-nodeid="1606"><strong data-nodeid="1894">map</strong>是一个十分常用的操作符，可用于从一种类型转换成另外一种类型，例如下面的例子，我把数值类型转换成字符串。程序执行的时候会打印 "String: 1" 和 "String: 2"。代码和图例如下所示。</p>
<pre class="lang-swift" data-nodeid="1607"><code data-language="swift"><span class="hljs-type">Observable</span>.of(<span class="hljs-number">1</span>, <span class="hljs-number">2</span>)
    .<span class="hljs-built_in">map</span> { <span class="hljs-string">"String: "</span> + <span class="hljs-type">String</span>($<span class="hljs-number">0</span>) }
    .subscribe(onNext: {
        <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
    })
    .disposed(by: disposeBag)
</code></pre>
<p data-nodeid="1608"><img src="https://s0.lgstatic.com/i/image6/M01/3C/07/Cgp9HWCH7R6ASEB0AAD5Z_BYeoQ092.png" alt="Drawing 7.png" data-nodeid="1897"></p>
<div data-nodeid="1609"><p style="text-align:center">转换操作符 map 的效果</p></div>
<p data-nodeid="1610"><strong data-nodeid="1906">compactMap</strong>常用于过滤掉值为<code data-backticks="1" data-nodeid="1902">nil</code>的操作符，你可以把 compactMap 理解为同时使用 filter 和 map 的两个操作符。filter 把<code data-backticks="1" data-nodeid="1904">nil</code>的值过滤掉，而 map 把非空的值进行转换。</p>
<p data-nodeid="1611">例如下面的例子中，我把字符串的值转换为数值类型，并把转换不成功的值过滤掉。由于 "not-a-number" 不能转换成数值类型，因此被过滤掉了，执行的时候会打印 1 和 2。代码示例如下所示：</p>
<pre class="lang-swift" data-nodeid="1612"><code data-language="swift"><span class="hljs-type">Observable</span>.of(<span class="hljs-string">"1"</span>, <span class="hljs-string">"not-a-number"</span>, <span class="hljs-string">"2"</span>)
    .<span class="hljs-built_in">compactMap</span> { <span class="hljs-type">Int</span>($<span class="hljs-number">0</span>) }
    .subscribe(onNext: {
        <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
    })
    .disposed(by: disposeBag)
</code></pre>
<p data-nodeid="1613"><img src="https://s0.lgstatic.com/i/image6/M01/3C/0F/CioPOWCH7SeAfVeYAAEfEP1ULSY822.png" alt="Drawing 9.png" data-nodeid="1914"></p>
<div data-nodeid="1614"><p style="text-align:center">转换操作符 compactMap 效果</p></div>
<p data-nodeid="1615"><strong data-nodeid="1919">flatMap</strong>用于把两层的 Observable 序列合并到一层。我们通过一个例子来解析到底怎样合并。</p>
<p data-nodeid="1616">请看代码示例：</p>
<pre class="lang-swift" data-nodeid="1617"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">TemperatureSensor</span> </span>{
  <span class="hljs-keyword">let</span> temperature: <span class="hljs-type">Observable</span>&lt;<span class="hljs-type">Int</span>&gt;
}
<span class="hljs-keyword">let</span> sensor1 = <span class="hljs-type">TemperatureSensor</span>(temperature: <span class="hljs-type">Observable</span>.of(<span class="hljs-number">21</span>, <span class="hljs-number">23</span>))
<span class="hljs-keyword">let</span> sensor2 = <span class="hljs-type">TemperatureSensor</span>(temperature: <span class="hljs-type">Observable</span>.of(<span class="hljs-number">22</span>, <span class="hljs-number">25</span>))
<span class="hljs-type">Observable</span>.of(sensor1, sensor2)
    .flatMap { $<span class="hljs-number">0</span>.temperature }
    .subscribe(onNext: {
        <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
    })
    .disposed(by: disposeBag)
</code></pre>
<p data-nodeid="1618">在这个例子中，我定义一个叫作<code data-backticks="1" data-nodeid="1922">TemperatureSensor</code>的结构体，用来表示收集温度的传感器，该结构体包含了一个类型为<code data-backticks="1" data-nodeid="1924">Observable</code>的<code data-backticks="1" data-nodeid="1926">temperature</code>的属性。</p>
<p data-nodeid="1619">假如天气站有多个这样的传感器，我们要把它们的温度信息合并到一个单独的 Observable 序列中方便统计，此时就可以使用 flatMap 来完成这项任务。</p>
<p data-nodeid="1620">具体来说，我们在<code data-backticks="1" data-nodeid="1930">flatMap</code>方法的闭包里面返回<code data-backticks="1" data-nodeid="1932">temperature</code>属性，由于该属性是一个<code data-backticks="1" data-nodeid="1934">Observable</code>对象，因此<code data-backticks="1" data-nodeid="1936">flatMap</code>方法会把这些序列统一合并到一个单独的 Observable 序列里面，并打印出 21、23、22、25。</p>
<p data-nodeid="1621"><img src="https://s0.lgstatic.com/i/image6/M01/3C/0F/CioPOWCH7TKAWC3hAAEPlMCt_uM223.png" alt="Drawing 11.png" data-nodeid="1940"></p>
<div data-nodeid="1622"><p style="text-align:center">转换操作符 flatMap 的效果</p></div>
<h4 data-nodeid="1623">合并操作符</h4>
<p data-nodeid="1624">合并操作符用于组装与合并多个 Observable 序列。我们通过 startWith，concat 和 merge 等几个常用的合并操作符，来看看它们是怎样运作的。</p>
<p data-nodeid="1625"><strong data-nodeid="1949">startWith</strong>可以使订阅者在接收到 Observable 序列的事件前，先收到传给 startWith 方法的事件。它的使用非常简单，例如在下面的例子中，我们把 3 和 4 传递给<code data-backticks="1" data-nodeid="1947">startWith</code>。那么在执行过程中，会先把 3 和 4 事件发送给订阅者，其运行效果为 3、4、1、2。代码示例如下：</p>
<pre class="lang-swift" data-nodeid="1626"><code data-language="swift"><span class="hljs-type">Observable</span>.of(<span class="hljs-number">1</span>, <span class="hljs-number">2</span>)
    .startWith(<span class="hljs-number">3</span>, <span class="hljs-number">4</span>)
    .subscribe(onNext: {
        <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
    })
    .disposed(by: disposeBag)
</code></pre>
<p data-nodeid="1627"><img src="https://s0.lgstatic.com/i/image6/M01/3C/0F/CioPOWCH7USAKIvoAADtwXiN318148.png" alt="Drawing 13.png" data-nodeid="1952"></p>
<div data-nodeid="1628"><p style="text-align:center">合并操作符 startWith 效果</p></div>
<p data-nodeid="1629"><strong data-nodeid="1961">日常中我们可以通过</strong><code data-backticks="1" data-nodeid="1956">startWith</code>方法，把加载事件插入网络数据事件之前，以此<strong data-nodeid="1962">来保持 UI 状态的自动更新。</strong></p>
<p data-nodeid="1630"><strong data-nodeid="1967">concat</strong>能把多个 Observable 序列按顺序合并在一起。例如，在下面的例子中我们合并了两个 Observable 序列，第一个包含 1 和 2，第二个包含 3 和 4，那么执行的时候会打印 1、2、3、4。代码示例如下。</p>
<pre class="lang-swift" data-nodeid="1631"><code data-language="swift"><span class="hljs-type">Observable</span>.of(<span class="hljs-number">1</span>, <span class="hljs-number">2</span>)
    .concat(<span class="hljs-type">Observable</span>.of(<span class="hljs-number">3</span>, <span class="hljs-number">4</span>))
    .subscribe(onNext: {
        <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
    })
    .disposed(by: disposeBag)
</code></pre>
<p data-nodeid="1632"><img src="https://s0.lgstatic.com/i/image6/M00/3C/0F/CioPOWCH7VeAeMD5AACnQDe-5Nk532.png" alt="Drawing 15.png" data-nodeid="1970"></p>
<div data-nodeid="1633"><p style="text-align:center">合并操作符 concat  效果</p></div>
<p data-nodeid="1634"><strong data-nodeid="1975">merge</strong>，常用于合并多个 Observable 序列的操作符，和 concat 不一样的地方是它能保持原来事件的顺序。我们可以通过一个例子来看看，它是怎样合并 Observable 序列的。代码示例如下：</p>
<pre class="lang-swift" data-nodeid="1635"><code data-language="swift"><span class="hljs-keyword">let</span> first = <span class="hljs-type">PublishSubject</span>&lt;<span class="hljs-type">Int</span>&gt;()
<span class="hljs-keyword">let</span> second = <span class="hljs-type">PublishSubject</span>&lt;<span class="hljs-type">Int</span>&gt;()
<span class="hljs-type">Observable</span>.of(first, second)
    .merge()
    .subscribe(onNext: {
        <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
    })
    .disposed(by: disposeBag)
first.onNext(<span class="hljs-number">1</span>)
first.onNext(<span class="hljs-number">2</span>)
second.onNext(<span class="hljs-number">11</span>)
first.onNext(<span class="hljs-number">3</span>)
second.onNext(<span class="hljs-number">12</span>)
second.onNext(<span class="hljs-number">13</span>)
first.onNext(<span class="hljs-number">4</span>)
</code></pre>
<p data-nodeid="1636">我们调用<code data-backticks="1" data-nodeid="1977">merge</code>方法把两个 PublishSubject 合并在一起，然后不同的 PublishSubject 会分别发出不同的<code data-backticks="1" data-nodeid="1979">next</code>事件，订阅者根据事件发生的顺序来接收到相关事件。如下图所示，程序执行时会打印 1、2、11、3、12、13、4。<br>
<img src="https://s0.lgstatic.com/i/image6/M00/3C/0F/CioPOWCH7W6AOGypAAEmk4CaMh0083.png" alt="Drawing 17.png" data-nodeid="1984"></p>
<div data-nodeid="1637"><p style="text-align:center">合并操作符 merge 的效果</p></div>
<p data-nodeid="1638"><strong data-nodeid="1989">combineLatest</strong>会把两个 Observable 序列里最后的事件合并起来，代码示例如下。</p>
<pre class="lang-swift" data-nodeid="1639"><code data-language="swift"><span class="hljs-keyword">let</span> first = <span class="hljs-type">PublishSubject</span>&lt;<span class="hljs-type">String</span>&gt;()
<span class="hljs-keyword">let</span> second = <span class="hljs-type">PublishSubject</span>&lt;<span class="hljs-type">String</span>&gt;()
<span class="hljs-type">Observable</span>.combineLatest(first, second) { $<span class="hljs-number">0</span> + $<span class="hljs-number">1</span> }
    .subscribe(onNext: {
        <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
    })
    .disposed(by: disposeBag)
first.onNext(<span class="hljs-string">"1"</span>)
second.onNext(<span class="hljs-string">"a"</span>)
first.onNext(<span class="hljs-string">"2"</span>)
second.onNext(<span class="hljs-string">"b"</span>)
second.onNext(<span class="hljs-string">"c"</span>)
first.onNext(<span class="hljs-string">"3"</span>)
first.onNext(<span class="hljs-string">"4"</span>)
</code></pre>
<p data-nodeid="1640">在程序执行过程中，当其中一个 PublishSubject 发出<code data-backticks="1" data-nodeid="1991">next</code>事件时，就会从另外一个 PublishSubject 取出其最后一个事件，然后调用<code data-backticks="1" data-nodeid="1993">combineLatest</code>方法的闭包，把这两个事件合并起来并通知订阅者。上述的例子在执行时会打印 1a、2a、2b、2c、3c、4c。</p>
<p data-nodeid="1641"><img src="https://s0.lgstatic.com/i/image6/M00/3C/0F/CioPOWCH7X6AJXQvAAEo9AcsIGo039.png" alt="Drawing 19.png" data-nodeid="1997"></p>
<div data-nodeid="1642"><p style="text-align:center">合并操作符 combineLatest</p></div>
<p data-nodeid="1643">在实际开发中，<code data-backticks="1" data-nodeid="1999">combineLatest</code>方法非常实用。我们可以用它来监听多个 Observable 序列，然后组合起来统一更新状态。例如在一个登录页面里面，我们可以同时监听用户名和密码两个输入框，当它们同时有值的时候才激活登录按钮。</p>
<p data-nodeid="1644"><strong data-nodeid="2005">zip</strong>也能用于合并两个 Observable 序列，和 combineLatest 不一样的地方是， zip 只会把两个 Observable 序列的事件配对合并。就像两队小朋友，排在前头的手牵手来到一个新队列。一旦出来就不再留在原有队列了。</p>
<p data-nodeid="1645">为了方便理解 zip 与 combineLatest 的区别，我在下面例子中也使用了一样的数据并保持事件发送的顺序。</p>
<pre class="lang-swift" data-nodeid="1646"><code data-language="swift"><span class="hljs-keyword">let</span> first = <span class="hljs-type">PublishSubject</span>&lt;<span class="hljs-type">String</span>&gt;()
<span class="hljs-keyword">let</span> second = <span class="hljs-type">PublishSubject</span>&lt;<span class="hljs-type">String</span>&gt;()
<span class="hljs-type">Observable</span>.<span class="hljs-built_in">zip</span>(first, second) { $<span class="hljs-number">0</span> + $<span class="hljs-number">1</span> }
    .subscribe(onNext: {
        <span class="hljs-built_in">print</span>($<span class="hljs-number">0</span>)
    })
    .disposed(by: disposeBag)
first.onNext(<span class="hljs-string">"1"</span>)
second.onNext(<span class="hljs-string">"a"</span>)
first.onNext(<span class="hljs-string">"2"</span>)
second.onNext(<span class="hljs-string">"b"</span>)
second.onNext(<span class="hljs-string">"c"</span>)
first.onNext(<span class="hljs-string">"3"</span>)
first.onNext(<span class="hljs-string">"4"</span>)
</code></pre>
<p data-nodeid="3066">在上述的例子中，有两个 PublishSubject，其中<code data-backticks="1" data-nodeid="3070">first</code>发出 1、2、3、4，而<code data-backticks="1" data-nodeid="3072">second</code>发出 a、b、c。<code data-backticks="1" data-nodeid="3074">zip</code>方法会返回它们的合并事件 1a、2b、3c。由于<code data-backticks="1" data-nodeid="3076">first</code>所发出<code data-backticks="1" data-nodeid="3078">next("4")</code>事件没有在<code data-backticks="1" data-nodeid="3080">second</code>里面找到对应的事件，所以合并后的 Observable 序列只有三个事件。</p>
<p data-nodeid="4099" class=""><img src="https://s0.lgstatic.com/i/image6/M01/3C/9B/Cgp9HWCLocCASVLAAAEE-3Z6-aU135.png" alt="图片5.png" data-nodeid="4102"></p><p style="text-align:center">合并操作符 zip 的效果</p><p></p>






<p data-nodeid="1650" class="">上面是常用的操作符，灵活使用它们，我们可以完成绝大部分的任务了。</p>
<h3 data-nodeid="1651" class="">总结</h3>
<p data-nodeid="1652">在这一讲中，我们介绍了 ViewModel 模式的架构与实现和 RxSwift 的操作符。有了 ViewModel，我们可以把业务逻辑从 View 层抽离出来，甚至把 View 层进行替换，例如把 UIKit 替换成 SwiftUI。而 UI 所需的数据，可以通过 ViewModel 模块把 Model 数据转换出来。至于转换工作，我们可以借助操作符来完成。</p>
<p data-nodeid="1653">有关本讲操作符的例子代码，我都放在项目中的<strong data-nodeid="2031">RxSwift Playground 文件</strong>里面，希望你能多练习，灵活运用。</p>
<p data-nodeid="1654">RxSwift 为我们提供了 50 多个操作符，我建议你到 rxmarbles.com 或者到 App Store 下载 RxMarbles App，并在 App 中替换各种参数来观察执行的结果，这样能帮助你学会所有的操作符，在现实工作中能选择合适的操作符来简化大量的开发工作。</p>
<p data-nodeid="1655"><strong data-nodeid="2036">思考题</strong></p>
<blockquote data-nodeid="1656">
<p data-nodeid="1657">请问你会把所有逻辑都编写在 ViewController 里面吗？如果没有，使用了怎样模式与架构来解耦呢？能分享一下这方面的经验吗？</p>
</blockquote>
<p data-nodeid="1658">请把你的想法写到留言区哦，下一讲我将介绍如何开发统一并且灵活的 UI。</p>
<p data-nodeid="1659"><strong data-nodeid="2042">源码地址：</strong></p>
<blockquote data-nodeid="1660">
<p data-nodeid="1661" class="">RxSwift Playground 文件地址：<br>
<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Playgrounds/RxSwiftPlayground.playground/Contents.swift?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2047">https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Playgrounds/RxSwiftPlayground.playground/Contents.swift</a><br>
ViewModel 协议定义的源码地址：<a href="https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Moments/Moments/Foundations/ViewModels?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2052">https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Moments/Moments/Foundations/ViewModels</a><br>
朋友圈功能 ViewModel 实现的源码地址：<br>
<a href="https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Moments/Moments/Features/Moments/ViewModels?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2058">https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Moments/Moments/Features/Moments/ViewModels</a></p>
</blockquote>

---

### 精选评论

##### **鸿：
> 留言++

##### **泽：
> viewModel.likes.forEach { let avatarURL = $0 let avatar: UIImageView = configure(.init()) { $0.translatesAutoresizingMaskIntoConstraints = false $0.asAvatar(cornerRadius: 2) $0.kf.setImage(with: avatarURL) } avatar.snp.makeConstraints { $0.width.equalTo(20) $0.height.equalTo(20) } likesStakeView.addArrangedSubview(avatar) }老师，看您的代码，这块有点没明白， 使用makeConstraints 不应该先加入view嘛，是因为stackview特殊嘛

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦哦，因为图片没有 intrinsicContentSize，所以我为它们配置了长度和高度。它们是通过 likesStakeView.addArrangedSubview(avatarImageView) 加入到父 StackView 的，你的理解是对的。

##### **泽：
> 老师， ReactiveObjc 有没有RXDataSource这样的东西

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我没有用过 ReactiveObjc，他里面好像就包括了和 UIKit 相关的内容。RxSwift 是为了可以跨平台，把 UIKit 抽到 RxCocoa，然后 RxSwiftCommunity 还提供 RxDataSources 等扩展。我觉得社区很好，所以一直使用。

##### **邪：
> 按照功能分离出多个mvc😂

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 也行的，只要符合设计原则把代码进行解耦就可以了。

##### **方：
> RxSwift 的确有点门槛

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的呀，为了解决学习曲线的问题，我在课程中把几个响应式编程的关键点讲述了，例如发送者，订阅者，操作符等等，希望通过这个系列能帮助大家可以入门。

##### Mr.Xu：
> 受益匪浅

##### *行：
> 老师，合并操作符 zip 的效果，这张图放错了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我刚才检查过，图是对的，结果是 1a，2b和3c

