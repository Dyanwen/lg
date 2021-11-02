<p data-nodeid="1852" class="">如今苹果公司力推的 SwiftUI 越来越流行，例如 Widget 等一些新功能只能使用 SwiftUI 进行开发，再加上 SwiftUI 又变得越来越稳定，可以说现在是学习和使用 SwiftUI 的良好时机。但并不是每个 App 都可以很方便地升级技术栈，幸运的是，Moments App 使用了 MVVM 的架构，该架构为我们提供了良好的灵活性和可扩展性，下面我们一起看看如何把 Moments App 的 UI 层从 UIKit 替换成 SwiftUI。</p>
<p data-nodeid="1853">在前面<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=657&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6669&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1955">第 16 讲</a>里，我们讲了如何使用 MVVM 模式来架构 Moments App。在这一讲中，我准备把 UIViewController 和 UIView 从 View 层移除，替换成 SwiftUI 的实现，如下图所示：</p>
<p data-nodeid="1854"><img src="https://s0.lgstatic.com/i/image6/M01/44/1B/Cgp9HWC90WWALsfHAAMRfIFPUjA184.png" alt="Drawing 0.png" data-nodeid="1959"></p>
<p data-nodeid="1855">可以看到，除了 View 层以外，其他模块（包括 ViewModel 和 Model 层等）都没有做任何的改动。下面我们就来剖析下这个实现原理和步骤。</p>
<h3 data-nodeid="1856">SwiftUI 的状态管理</h3>
<p data-nodeid="1857">SwiftUI 是一个由状态驱动的 UI 框架，为了更好地理解 SwiftUI 的使用，我们就先来看看 SwiftUI 是如何管理状态的。</p>
<p data-nodeid="1858"><strong data-nodeid="1967">状态管理最简单的方式是使用 @State 属性包装器（Property Wrapper）</strong>，下面是使用 @State 的示例代码：</p>
<pre class="lang-swift" data-nodeid="1859"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">ContentView</span>: <span class="hljs-title">View</span> </span>{
    @<span class="hljs-type">State</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> age = <span class="hljs-number">20</span>
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-type">Button</span>(<span class="hljs-string">"生日啦，现在几岁: \(age)"</span>) {
            age += <span class="hljs-number">1</span>
        }
    }
}
</code></pre>
<p data-nodeid="1860">我们在<code data-backticks="1" data-nodeid="1969">ContentView</code>里面创建了一个名叫<code data-backticks="1" data-nodeid="1971">age</code>的属性，由于使用了 @State 属性包装器，所以 SwiftUI 会帮我们自动管理这个属性的内存并监听其状态更新的情况。在上述的例子中，当用户点击“生日啦”按钮时，就会把<code data-backticks="1" data-nodeid="1973">age</code>属性的值增加一，这一更改会促使 SwiftUI 自动刷新<code data-backticks="1" data-nodeid="1975">ContentView</code>。</p>
<p data-nodeid="1861">@State 适合为某个特定的 View 管理类型为值（Value）的属性，而且我们通常把 @State 的属性都定义为<code data-backticks="1" data-nodeid="1978">private</code>（私有的）以禁止外部的访问。但如何实现多个对象间（例如，父子视图间）的状态共享呢？那就需要使用到 @StateObject 和 @ObservedObject 属性包装器了。这两个属性包装器所定义的属性都必须遵循<code data-backticks="1" data-nodeid="1980">ObservableObject</code>协议。</p>
<p data-nodeid="1862">那接下来我们就再看一下为什么使用<code data-backticks="1" data-nodeid="1983">ObservableObject</code>协议吧。</p>
<p data-nodeid="12378" class="te-preview-highlight"><strong data-nodeid="12383">为了让 SwiftUI 能访问来自 Model 的状态更新，我们必须让 Model 遵循 ObservableObject 协议</strong>。那 Model 怎样才能发送状态通知呢？可以结合下面的例子来理解。</p>









<pre class="lang-swift" data-nodeid="1864"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserObservableObject</span>: <span class="hljs-title">ObservableObject</span> </span>{
    <span class="hljs-keyword">var</span> name = <span class="hljs-string">"Jake"</span>
    <span class="hljs-keyword">var</span> age = <span class="hljs-number">20</span> {
        <span class="hljs-keyword">willSet</span> {
            objectWillChange.send()
        }
    }
}
</code></pre>
<p data-nodeid="1865"><code data-backticks="1" data-nodeid="1995">UserObservableObject</code>是一个遵循了<code data-backticks="1" data-nodeid="1997">ObservableObject</code>协议的类。因为所有遵循<code data-backticks="1" data-nodeid="1999">ObservableObject</code>协议的子类型都必须是引用类型，所以我们只能使用类而不是结构体（Struct）。<code data-backticks="1" data-nodeid="2001">UserObservableObject</code>定义了两个属性：<code data-backticks="1" data-nodeid="2003">age</code>属性的<code data-backticks="1" data-nodeid="2005">willSet</code>里面调用了<code data-backticks="1" data-nodeid="2007">objectWillChange.send()</code>方法，当我们修改<code data-backticks="1" data-nodeid="2009">age</code>属性时，就会发送状态更新通知；而<code data-backticks="1" data-nodeid="2011">name</code>属性没有调用<code data-backticks="1" data-nodeid="2013">objectWillChange.send()</code>方法，因此我们修改它的时候并不会发送更新通知。</p>
<p data-nodeid="1866">你可以看到，所有需要发送更新通知的属性都必须编写重复的<code data-backticks="1" data-nodeid="2016">willSet</code>代码，幸运的是苹果为我们提供了 <code data-backticks="1" data-nodeid="2018">@Published</code>属性包装器来简化编写更新通知的工作。有了<code data-backticks="1" data-nodeid="2020">@Published</code>，上述的代码就可以简化为如下：</p>
<pre class="lang-swift" data-nodeid="1867"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserObservableObject</span>: <span class="hljs-title">ObservableObject</span> </span>{
    <span class="hljs-keyword">var</span> name = <span class="hljs-string">"Jake"</span>
    @<span class="hljs-type">Published</span> <span class="hljs-keyword">var</span> age = <span class="hljs-number">20</span>
}
</code></pre>
<p data-nodeid="1868">我们只需要在发送状态更新的属性定义前加上<code data-backticks="1" data-nodeid="2023">@Published</code>即可。</p>
<p data-nodeid="1869">介绍完<code data-backticks="1" data-nodeid="2026">ObservableObject</code>协议以后，我们就可以通过下面的例子看看如何使用 @StateObject 和 @ObservedObject 属性包装器了。</p>
<pre class="lang-swift" data-nodeid="1870"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">ChildView</span>: <span class="hljs-title">View</span> </span>{
    @<span class="hljs-type">ObservedObject</span> <span class="hljs-keyword">var</span> user: <span class="hljs-type">UserObservableObject</span>
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-type">Button</span>(<span class="hljs-string">"生日啦，现在几岁: \(user.age)"</span>) {
            user.age += <span class="hljs-number">1</span>
        }
    }
}
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">ParentView</span>: <span class="hljs-title">View</span> </span>{
    @<span class="hljs-type">StateObject</span> <span class="hljs-keyword">var</span> user: <span class="hljs-type">UserObservableObject</span> = .<span class="hljs-keyword">init</span>()
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-type">VStack</span> {
            <span class="hljs-type">Text</span>(<span class="hljs-string">"你的名字：\(user.name)"</span>)
            <span class="hljs-type">ChildView</span>(user: user)
        }
    }
}
</code></pre>
<p data-nodeid="1871"><strong data-nodeid="2047">@StateObject 和 @ObservedObject 都可以定义用于状态共享的属性，而且这些属性的类型都必须遵循</strong><code data-backticks="1" data-nodeid="2031">ObservableObject</code>协议。不同的地方是 @StateObject 用于生成和管理状态属性的生命周期，而 @ObservedObject 只能把共享状态从外部传递进来。例如，在上面的示例代码中，我们在<code data-backticks="1" data-nodeid="2033">ParentView</code>里使用 @StateObject 来定义并初始化<code data-backticks="1" data-nodeid="2035">user</code>属性，然后传递给<code data-backticks="1" data-nodeid="2037">ChildView</code>的<code data-backticks="1" data-nodeid="2039">user</code>属性。由于<code data-backticks="1" data-nodeid="2041">ChildView</code>的<code data-backticks="1" data-nodeid="2043">user</code>属性来自外部的<code data-backticks="1" data-nodeid="2045">ParentView</code>，因此定义为 @ObservedObject。</p>
<p data-nodeid="1872">当我们需要共享状态的时候，通常在父对象里定义和初始化一个 @StateObject 属性，然后传递给子对象里的 @ObservedObject 属性。如果只有两层关系还是很方便的，但假如有好几层的父子关系，逐层传递会变得非常麻烦，那有没有好办法解决这个问题呢？</p>
<p data-nodeid="1873">@EnvironmentObject 就是用于解决这个问题的。@EnvironmentObject 能帮我们把状态共享到整个 App 里面，下面还是通过一个例子来看看。</p>
<pre class="lang-swift" data-nodeid="1874"><code data-language="swift">@main
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">MomentsApp</span>: <span class="hljs-title">App</span> </span>{
    @<span class="hljs-type">StateObject</span> <span class="hljs-keyword">var</span> user: <span class="hljs-type">UserObservableObject</span> = .<span class="hljs-keyword">init</span>()
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">Scene</span> {
        <span class="hljs-type">WindowGroup</span> {
            <span class="hljs-type">ParentView</span>()
                .environmentObject(user)
        }
    }
}
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">ChildView</span>: <span class="hljs-title">View</span> </span>{
    @<span class="hljs-type">EnvironmentObject</span> <span class="hljs-keyword">var</span> user: <span class="hljs-type">UserObservableObject</span>
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-type">Button</span>(<span class="hljs-string">"生日啦，现在几岁: \(user.age)"</span>) {
            user.age += <span class="hljs-number">1</span>
        }
    }
}
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">ParentView</span>: <span class="hljs-title">View</span> </span>{
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-type">VStack</span> {
            <span class="hljs-type">ChildView</span>()
        }
    }
}
</code></pre>
<p data-nodeid="1875">我们在<code data-backticks="1" data-nodeid="2051">MomentsApp</code>里面通过 @StateObject 定义并初始化<code data-backticks="1" data-nodeid="2053">user</code>属性，然后调用<code data-backticks="1" data-nodeid="2055">environmentObject()</code>方法把该属性注册成环境对象。<code data-backticks="1" data-nodeid="2057">MomentsApp</code>内嵌了<code data-backticks="1" data-nodeid="2059">ParentView</code>，而<code data-backticks="1" data-nodeid="2061">ParentView</code>并没有使用<code data-backticks="1" data-nodeid="2063">user</code>属性。<code data-backticks="1" data-nodeid="2065">ParentView</code>内嵌了<code data-backticks="1" data-nodeid="2067">ChildView</code>，<code data-backticks="1" data-nodeid="2069">ChildView</code>则通过 @EnvironmentObject 来定义<code data-backticks="1" data-nodeid="2071">user</code>属性，这样<code data-backticks="1" data-nodeid="2073">ChildView</code>就能从环境对象中取出<code data-backticks="1" data-nodeid="2075">MomentsApp</code>注册的值了。</p>
<p data-nodeid="1876"><strong data-nodeid="2081">@EnvironmentObject 能帮我们把对象传递到 App 任何的地方，特别适合共享公共的状态</strong>，例如用户登录的信息等。但是 @EnvironmentObject 有点像 Singleton，我们不能过度使用它，否则会增加模块间的耦合度。</p>
<p data-nodeid="1877">@ObservedObject 与 @EnvironmentObject 都能帮助我们共享引用类型的属性，但如何共享值类型的属性呢？<strong data-nodeid="2087">@Binding 属性包装器就能帮我们定义共享值类型的属性。</strong> 下面我们还是通过示例代码来看看如何使用 @Binding。</p>
<pre class="lang-swift" data-nodeid="1878"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">ChildView</span>: <span class="hljs-title">View</span> </span>{
    @<span class="hljs-type">Binding</span> <span class="hljs-keyword">var</span> isPresented: <span class="hljs-type">Bool</span>
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-type">Button</span>(<span class="hljs-string">"关闭"</span>) {
            isPresented = <span class="hljs-literal">false</span>
        }
    }
}
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">ParentView</span>: <span class="hljs-title">View</span> </span>{
    @<span class="hljs-type">State</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> showingChildView = <span class="hljs-literal">false</span>
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-type">VStack</span> {
            <span class="hljs-type">Text</span>(<span class="hljs-string">"父 View"</span>)
        }.sheet(isPresented: $showingChildView) {
            <span class="hljs-type">ChildView</span>(isPresented: $showingChildView)
        }
    }
}
</code></pre>
<p data-nodeid="1879"><code data-backticks="1" data-nodeid="2088">ChildView</code>通过 @Binding 定义了<code data-backticks="1" data-nodeid="2090">isPresented</code>属性，表示该视图是否可见。该属性的值与<code data-backticks="1" data-nodeid="2092">ParentView</code>的<code data-backticks="1" data-nodeid="2094">showingChildView</code>属性同步。通过 @Binding，我们就可以把值类型的属性进行共享了。</p>
<p data-nodeid="1880">至此，我们就介绍完 SwiftUI 的状态管理了。</p>
<h3 data-nodeid="1881">SwiftUI 的架构与实现</h3>
<p data-nodeid="1882">下面一起来看看使用 SwiftUI 开发 View 层的系统架构图。</p>
<p data-nodeid="1883"><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC90amACsvfAAHhOfaicTI452.png" alt="Drawing 1.png" data-nodeid="2101"></p>
<p data-nodeid="1884">该架构图由两部分组成，分别是左边的 View 模块和右边的 ViewModel 模块。由于 View 模块依赖了 ViewModel 模块，所以这里我们就先看右边的 ViewModel 模块。该模块包含了<code data-backticks="1" data-nodeid="2103">MomentsTimelineViewModel</code>、<code data-backticks="1" data-nodeid="2105">ListItemViewModel</code>、<code data-backticks="1" data-nodeid="2107">MomentListItemViewModel</code>和<code data-backticks="1" data-nodeid="2109">UserProfileListItemViewModel</code>四个原有的 ViewModel，因为它们具有良好的可扩展性，所以我们无须对它们进行任何的改动。</p>
<h4 data-nodeid="1885">1. 桥接 RxSwift 与 SwiftUI</h4>
<p data-nodeid="1886">为了把这些 ViewModel 类型桥接到 SwiftUI 版本的 View 模块，我们增加了两个类型：<code data-backticks="1" data-nodeid="2115">MomentsListObservableObject</code>和<code data-backticks="1" data-nodeid="2117">IdentifiableListItemViewModel</code>。<code data-backticks="1" data-nodeid="2119">MomentsListObservableObject</code>负责给 SwiftUI 组件发送更新消息，下面是它的具体实现：</p>
<pre class="lang-swift" data-nodeid="1887"><code data-language="swift"><span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MomentsListObservableObject</span>: <span class="hljs-title">ObservableObject</span> </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">let</span> viewModel: <span class="hljs-type">MomentsTimelineViewModel</span>
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">let</span> disposeBag: <span class="hljs-type">DisposeBag</span> = .<span class="hljs-keyword">init</span>()
    @<span class="hljs-type">Published</span> <span class="hljs-keyword">var</span> listItems: [<span class="hljs-type">IdentifiableListItemViewModel</span>] = []
    <span class="hljs-keyword">init</span>(userID: <span class="hljs-type">String</span>, momentsRepo: <span class="hljs-type">MomentsRepoType</span>) {
        viewModel = <span class="hljs-type">MomentsTimelineViewModel</span>(userID: userID, momentsRepo: momentsRepo)
        setupBindings()
    }
    <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">loadItems</span><span class="hljs-params">()</span></span> {
        viewModel.loadItems()
            .subscribe()
            .disposed(by: disposeBag)
    }
    <span class="hljs-keyword">private</span> <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">setupBindings</span><span class="hljs-params">()</span></span> {
        viewModel.listItems
            .observeOn(<span class="hljs-type">MainScheduler</span>.instance)
            .subscribe(onNext: { [<span class="hljs-keyword">weak</span> <span class="hljs-keyword">self</span>] items <span class="hljs-keyword">in</span>
                <span class="hljs-keyword">guard</span> <span class="hljs-keyword">let</span> <span class="hljs-keyword">self</span> = <span class="hljs-keyword">self</span> <span class="hljs-keyword">else</span> { <span class="hljs-keyword">return</span> }
                <span class="hljs-keyword">self</span>.listItems.removeAll()
                <span class="hljs-keyword">self</span>.listItems.append(contentsOf: items.flatMap { $<span class="hljs-number">0</span>.items }.<span class="hljs-built_in">map</span> { <span class="hljs-type">IdentifiableListItemViewModel</span>(viewModel: $<span class="hljs-number">0</span>) })
            })
            .disposed(by: disposeBag)
    }
}
</code></pre>
<p data-nodeid="1888"><code data-backticks="1" data-nodeid="2121">MomentsListObservableObject</code>遵循了<code data-backticks="1" data-nodeid="2123">ObservableObject</code>协议，并使用了 @Published 来定义<code data-backticks="1" data-nodeid="2125">listItems</code>属性，这样使得<code data-backticks="1" data-nodeid="2127">listItems</code>的状态更新会自动往外发送。<br>
<code data-backticks="1" data-nodeid="2130">listItems</code>属性的类型是<code data-backticks="1" data-nodeid="2132">IdentifiableListItemViewModel</code>的数组，下面是<code data-backticks="1" data-nodeid="2134">IdentifiableListItemViewModel</code>的具体实现：</p>
<pre class="lang-swift" data-nodeid="1889"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">IdentifiableListItemViewModel</span>: <span class="hljs-title">Identifiable</span> </span>{
    <span class="hljs-keyword">let</span> id: <span class="hljs-type">UUID</span> = .<span class="hljs-keyword">init</span>()
    <span class="hljs-keyword">let</span> viewModel: <span class="hljs-type">ListItemViewModel</span>
}
</code></pre>
<p data-nodeid="1890"><code data-backticks="1" data-nodeid="2136">IdentifiableListItemViewModel</code>其实是<code data-backticks="1" data-nodeid="2138">ListItemViewModel</code>的一个包装类型，因为我们要在 SwiftUI 上重复显示<code data-backticks="1" data-nodeid="2140">ListItemViewModel</code>的数据，所以就要用到<code data-backticks="1" data-nodeid="2142">ForEach</code>语句来执行循环操作。而<code data-backticks="1" data-nodeid="2144">ForEach</code>语句要求所有 Model 类型都遵循<code data-backticks="1" data-nodeid="2146">Identifiable</code>协议，因此，我们定义了<code data-backticks="1" data-nodeid="2148">IdentifiableListItemViewModel</code>来遵循<code data-backticks="1" data-nodeid="2150">Identifiable</code>协议，并把<code data-backticks="1" data-nodeid="2152">ListItemViewModel</code>包装在里面，同时还通过<code data-backticks="1" data-nodeid="2154">id</code>属性来返回一个 UUID 的实例。</p>
<p data-nodeid="1891">在<code data-backticks="1" data-nodeid="2157">init()</code>初始化函数里，我们订阅了<code data-backticks="1" data-nodeid="2159">MomentsTimelineViewModel</code>的<code data-backticks="1" data-nodeid="2161">listItems</code>Subject 属性的更新，而且把接收到的数据转换成<code data-backticks="1" data-nodeid="2163">IdentifiableListItemViewModel</code>类型并赋值给<code data-backticks="1" data-nodeid="2165">listItems</code>属性，这样就能把 RxSwift 的事件消息桥接给 SwiftUI 进行自动更新了。</p>
<p data-nodeid="1892">接着再来看看 View 模块，该模块由<code data-backticks="1" data-nodeid="2168">SwiftUIMomentsTimelineView</code>、<code data-backticks="1" data-nodeid="2170">SwiftUIMomentsListItemView</code>、<code data-backticks="1" data-nodeid="2172">SwiftUIMomentListItemView</code>和<code data-backticks="1" data-nodeid="2174">SwiftUIUserProfileListItemView</code>所组成，你可以结合下图了解它们之间的嵌套关系。</p>
<p data-nodeid="1893"><img src="https://s0.lgstatic.com/i/image6/M01/44/1B/Cgp9HWC90b-AAGkCAAdM0BD6GgE546.png" alt="Drawing 2.png" data-nodeid="2178"></p>
<p data-nodeid="1894"><code data-backticks="1" data-nodeid="2179">SwiftUIMomentsTimelineView</code>是一个容器视图，包含了多个<code data-backticks="1" data-nodeid="2181">SwiftUIMomentsListItemView</code>。<code data-backticks="1" data-nodeid="2183">SwiftUIMomentsListItemView</code>会根据 ViewModel 的具体类型来显示<code data-backticks="1" data-nodeid="2185">SwiftUIUserProfileListItemView</code>或者<code data-backticks="1" data-nodeid="2187">SwiftUIMomentListItemView</code>。</p>
<h4 data-nodeid="1895">2. 朋友圈时间轴视图</h4>
<p data-nodeid="1896">下面我们分别看看它们的实现吧，首先看容器视图<code data-backticks="1" data-nodeid="2193">SwiftUIMomentsTimelineView</code>的代码实现。</p>
<pre class="lang-swift" data-nodeid="1897"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">SwiftUIMomentsTimelineView</span>: <span class="hljs-title">View</span> </span>{
    @<span class="hljs-type">StateObject</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> userDataStore: <span class="hljs-type">UserDataStoreObservableObject</span> = .<span class="hljs-keyword">init</span>()
    @<span class="hljs-type">StateObject</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> momentsList: <span class="hljs-type">MomentsListObservableObject</span> = .<span class="hljs-keyword">init</span>(userID: <span class="hljs-type">UserDataStore</span>.current.userID, momentsRepo: <span class="hljs-type">MomentsRepo</span>.shared)
    @<span class="hljs-type">State</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> isDragging: <span class="hljs-type">Bool</span> = <span class="hljs-literal">false</span>
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-type">ScrollView</span>(axes, showsIndicators: <span class="hljs-literal">true</span>) {
            <span class="hljs-type">LazyVStack</span> {
                <span class="hljs-type">ForEach</span> (momentsList.listItems) { item <span class="hljs-keyword">in</span>
                    <span class="hljs-type">SwiftUIMomentsListItemView</span>(viewModel: item.viewModel, isDragging: $isDragging).ignoresSafeArea(.all)
                }.onAppear(perform: {
                    momentsList.loadItems()
                })
            }
        }.frame(minWidth: <span class="hljs-number">0</span>, maxWidth: .infinity, minHeight: <span class="hljs-number">0</span>, maxHeight: .infinity)
        .background(<span class="hljs-type">Color</span>(<span class="hljs-string">"background"</span>))
        .ignoresSafeArea(.all)
        .environmentObject(userDataStore)
    }
}
</code></pre>
<p data-nodeid="1898">我们使用 @StateObject 定义了<code data-backticks="1" data-nodeid="2196">userDataStore</code>属性，并通过<code data-backticks="1" data-nodeid="2198">environmentObject()</code>方法把它注册到环境对象中，这样就使得所有的子视图都能通过 @EnvironmentObject 来访问<code data-backticks="1" data-nodeid="2200">userDataStore</code>属性的值了。</p>
<p data-nodeid="1899"><code data-backticks="1" data-nodeid="2202">SwiftUIMomentsTimelineView</code>的布局比较简单，是一个<code data-backticks="1" data-nodeid="2204">ScrollView</code>，在<code data-backticks="1" data-nodeid="2206">ScrollView</code>里通过<code data-backticks="1" data-nodeid="2208">LazyVStack</code>和<code data-backticks="1" data-nodeid="2210">ForEach</code>把<code data-backticks="1" data-nodeid="2212">momentsList.listItems</code>的每一条数据通过<code data-backticks="1" data-nodeid="2214">SwiftUIMomentsListItemView</code>分别显示出来，而且在初始化<code data-backticks="1" data-nodeid="2216">SwiftUIMomentsListItemView</code>的时候把具体的 ViewModel 以及<code data-backticks="1" data-nodeid="2218">isDragging</code>属性传递进去。</p>
<h4 data-nodeid="1900">3. 中介视图</h4>
<p data-nodeid="1901"><code data-backticks="1" data-nodeid="2223">SwiftUIMomentsListItemView</code>担任中介的角色，其具体代码实现如下：</p>
<pre class="lang-swift" data-nodeid="1902"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">SwiftUIMomentsListItemView</span>: <span class="hljs-title">View</span> </span>{
    <span class="hljs-keyword">let</span> viewModel: <span class="hljs-type">ListItemViewModel</span>
    @<span class="hljs-type">Binding</span> <span class="hljs-keyword">var</span> isDragging: <span class="hljs-type">Bool</span>
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> viewModel = viewModel <span class="hljs-keyword">as</span>? <span class="hljs-type">UserProfileListItemViewModel</span> {
            <span class="hljs-type">SwiftUIUserProfileListItemView</span>(viewModel: viewModel, isDragging: $isDragging)
        } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> viewModel = viewModel <span class="hljs-keyword">as</span>? <span class="hljs-type">MomentListItemViewModel</span> {
            <span class="hljs-type">SwiftUIMomentListItemView</span>(viewModel: viewModel)
        }
    }
}
</code></pre>
<p data-nodeid="1903">我们使用了 @Binding 来定义<code data-backticks="1" data-nodeid="2226">isDragging</code>属性，这样就能与父视图<code data-backticks="1" data-nodeid="2228">SwiftUIMomentsTimelineView</code>共享用户的拖动状态了。<code data-backticks="1" data-nodeid="2230">SwiftUIMomentsListItemView</code>本身不做任何的显示操作，而是在<code data-backticks="1" data-nodeid="2232">body</code>属性里根据<code data-backticks="1" data-nodeid="2234">viewModel</code>的类型来分别通过<code data-backticks="1" data-nodeid="2236">SwiftUIUserProfileListItemView</code>或者<code data-backticks="1" data-nodeid="2238">SwiftUIMomentListItemView</code>进行显示。为什么需要这样做呢？因为 SwiftUI 里所有的组件都是值类型，例如 View 就不支持继承关系，我们无法使用多态（Polymorphism）的方式来动态显示的子 View，只能通过条件判断语句来选择性显示不同的 View。</p>
<h4 data-nodeid="1904">4. 用户属性视图</h4>
<p data-nodeid="1905">朋友圈功能最上面的部分是用户属性视图，下面我们看一下它的具体实现。由于<code data-backticks="1" data-nodeid="2244">SwiftUIUserProfileListItemView</code>的具体实现代码有点长，所以这里我把它拆成几部分来分别解释。</p>
<pre class="lang-swift" data-nodeid="1906"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">SwiftUIUserProfileListItemView</span>: <span class="hljs-title">View</span> </span>{
    <span class="hljs-keyword">let</span> viewModel: <span class="hljs-type">UserProfileListItemViewModel</span>
    @<span class="hljs-type">Binding</span> <span class="hljs-keyword">var</span> isDragging: <span class="hljs-type">Bool</span>
    @<span class="hljs-type">State</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> viewSize: <span class="hljs-type">CGSize</span> = .zero
}
</code></pre>
<p data-nodeid="1907">首先看一下属性的定义，我们定义了<code data-backticks="1" data-nodeid="2247">viewModel</code>属性来保存从父视图传进来的<code data-backticks="1" data-nodeid="2249">UserProfileListItemViewModel</code>对象，这样我们就能使用该<code data-backticks="1" data-nodeid="2251">viewModel</code>里的属性来进行显示了。</p>
<p data-nodeid="1908">同时我们还使用了 @Binding 来定义<code data-backticks="1" data-nodeid="2254">isDragging</code>属性，该属性与父视图<code data-backticks="1" data-nodeid="2256">SwiftUIMomentsTimelineView</code>共享用户拖动的状态。有了这个属性，我们在启动触摸动画时就可以停止父视图的拖动事件，从而避免奇怪的拖动效果。</p>
<p data-nodeid="1909">另外，我们还使用 @State 来定义一个私有的属性<code data-backticks="1" data-nodeid="2259">viewSize</code>，该属性用于控制拖拉动画的视图大小。</p>
<p data-nodeid="1910">为了更好地理解布局的代码实现，我们可以结合下面的图来看看各个组件之间的嵌套关系。</p>
<p data-nodeid="1911"><img src="https://s0.lgstatic.com/i/image6/M01/44/1B/Cgp9HWC90d-AH97-AAYCP8Dw-HE675.png" alt="Drawing 3.png" data-nodeid="2264"></p>
<p data-nodeid="1912">因为我们要把名字和头像放在底部，所以使用了用于垂直布局的<code data-backticks="1" data-nodeid="2266">VStack</code>。在该<code data-backticks="1" data-nodeid="2268">VStack</code>里先放一个<code data-backticks="1" data-nodeid="2270">Spacer</code>，这样能把下面的<code data-backticks="1" data-nodeid="2272">HStack</code>压到底部。<code data-backticks="1" data-nodeid="2274">HStack</code>用于水平布局，我们可以通过<code data-backticks="1" data-nodeid="2276">Spacer</code>把其他视图推到右边，右边是用于显示名字的<code data-backticks="1" data-nodeid="2278">Text</code>和显示头像的<code data-backticks="1" data-nodeid="2280">KFImage</code>控件。这所有的布局代码都存放在<code data-backticks="1" data-nodeid="2282">body</code>属性里，如下所示：</p>
<pre class="lang-swift" data-nodeid="1913"><code data-language="swift"><span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
    <span class="hljs-type">VStack</span> {
        <span class="hljs-type">Spacer</span>()
        <span class="hljs-type">HStack</span> {
            <span class="hljs-type">Spacer</span>()
            <span class="hljs-type">Text</span>(viewModel.name)
                .font(.title2)
                .foregroundColor(.white)
                .padding(.trailing, <span class="hljs-number">10</span>)
            <span class="hljs-type">KFImage</span>(viewModel.avatarURL)
                .resizable()
                .aspectRatio(contentMode: .fill)
                .frame(width: <span class="hljs-number">80</span>, height: <span class="hljs-number">80</span>, alignment: .center)
                .clipShape(<span class="hljs-type">Circle</span>())
        }
        .padding(.trailing, <span class="hljs-number">10</span>)
        .padding(.bottom, <span class="hljs-number">10</span>)
    }
    .frame(height: <span class="hljs-number">350</span>)
    .frame(maxWidth: .infinity)
}
</code></pre>
<p data-nodeid="1914">由于<code data-backticks="1" data-nodeid="2285">Spacer</code>不能提供高度和宽度，所以除了布局代码以外，我们还需要调用<code data-backticks="1" data-nodeid="2287">frame(height: 350)</code>方法来配置视图的高度，然后使用<code data-backticks="1" data-nodeid="2289">frame(maxWidth: .infinity)</code>方法使得视图占据设备的全部宽度。</p>
<p data-nodeid="1915">你可能会问，后面两个深蓝色的圆圈和背景图在哪里配置呢？其实它们都放在<code data-backticks="1" data-nodeid="2292">background</code>方法里面，具体代码如下：</p>
<pre class="lang-swift" data-nodeid="1916"><code data-language="swift">.background(
    <span class="hljs-type">ZStack</span> {
        <span class="hljs-type">Image</span>(uiImage: #imageLiteral(resourceName: <span class="hljs-string">"Blob"</span>))
            .offset(x: -<span class="hljs-number">200</span>, y: -<span class="hljs-number">200</span>)
            .rotationEffect(<span class="hljs-type">Angle</span>(degrees: <span class="hljs-number">450</span>))
            .blendMode(.plusDarker)
        <span class="hljs-type">Image</span>(uiImage: #imageLiteral(resourceName: <span class="hljs-string">"Blob"</span>))
            .offset(x: -<span class="hljs-number">200</span>, y: -<span class="hljs-number">250</span>)
            .rotationEffect(<span class="hljs-type">Angle</span>(degrees: <span class="hljs-number">360</span>), anchor: .leading)
            .blendMode(.overlay)
    }
)
.background(
    <span class="hljs-type">KFImage</span>(viewModel.backgroundImageURL)
        .resizable()
        .offset(x: viewSize.width / <span class="hljs-number">20</span>, y: viewSize.height / <span class="hljs-number">20</span>)
)
.clipShape(<span class="hljs-type">RoundedRectangle</span>(cornerRadius: <span class="hljs-number">30</span>, style: .continuous))
</code></pre>
<p data-nodeid="1917">这里调用了两次<code data-backticks="1" data-nodeid="2295">background</code>方法。在第一个<code data-backticks="1" data-nodeid="2297">background</code>方法里，我们使用了<code data-backticks="1" data-nodeid="2299">ZStack</code>来进行布局，<code data-backticks="1" data-nodeid="2301">ZStack</code>能帮助我们布局彼此覆盖的视图。在<code data-backticks="1" data-nodeid="2303">ZStack</code>里，我们存放了两个名叫 Blob 的<code data-backticks="1" data-nodeid="2305">Image</code>组件，由于它们使用了不一样的<code data-backticks="1" data-nodeid="2307">blendMode</code>，所以显示的效果有所不同。</p>
<p data-nodeid="1918">在第二个<code data-backticks="1" data-nodeid="2310">background</code>方法里，我们使用了<code data-backticks="1" data-nodeid="2312">KFImage</code>来加载背景图片，同时把<code data-backticks="1" data-nodeid="2314">viewSize</code>传递给<code data-backticks="1" data-nodeid="2316">offset()</code>方法来实现非常微妙的视差（parallax）效果。</p>
<p data-nodeid="1919">最后我们调用了<code data-backticks="1" data-nodeid="2319">clipShape()</code>方法来配置大圆角的效果，这是近期一种流行的设计风格。</p>
<p data-nodeid="1920">以上都是配置静态 UI 风格的代码，下面我们再来看看如何为<code data-backticks="1" data-nodeid="2322">SwiftUIUserProfileListItemView</code>呈现浮动的动画效果，如下实现代码：</p>
<pre class="lang-swift" data-nodeid="1921"><code data-language="swift">.scaleEffect(isDragging ? <span class="hljs-number">0.9</span> : <span class="hljs-number">1</span>)
.animation(.timingCurve(<span class="hljs-number">0.2</span>, <span class="hljs-number">0.8</span>, <span class="hljs-number">0.2</span>, <span class="hljs-number">1</span>, duration: <span class="hljs-number">0.8</span>))
.rotation3DEffect(<span class="hljs-type">Angle</span>(degrees: <span class="hljs-number">5</span>), axis: (x: viewSize.width, y: viewSize.height, z: <span class="hljs-number">0</span>))
.gesture(
    <span class="hljs-type">DragGesture</span>().onChanged({ value <span class="hljs-keyword">in</span>
        <span class="hljs-keyword">self</span>.isDragging = <span class="hljs-literal">true</span>
        <span class="hljs-keyword">self</span>.viewSize = value.translation
    }).onEnded({ <span class="hljs-number">_</span> <span class="hljs-keyword">in</span>
        <span class="hljs-keyword">self</span>.isDragging = <span class="hljs-literal">false</span>
        <span class="hljs-keyword">self</span>.viewSize = .zero
    })
)
</code></pre>
<p data-nodeid="1922">当调用<code data-backticks="1" data-nodeid="2325">scaleEffect()</code>方法时，我们根据<code data-backticks="1" data-nodeid="2327">isDragging</code>属性的状态来配置不同的缩放系数，这样能使得当用户拖拉视图时，视图会变小一点点。然后调用<code data-backticks="1" data-nodeid="2329">animation()</code>方法使得视图改变大小时会有平滑的转换动画效果，<code data-backticks="1" data-nodeid="2331">rotation3DEffect()</code>方法会使得拖拉视图时有浮动效果，<code data-backticks="1" data-nodeid="2333">gesture()</code>方法让我们可以根据用户的触摸状态来改变<code data-backticks="1" data-nodeid="2335">isDragging</code>和<code data-backticks="1" data-nodeid="2337">viewSize</code>的状态，从而影响动画的运行状态。</p>
<h4 data-nodeid="1923">5. 朋友圈信息视图</h4>
<p data-nodeid="1924">看完用户属性视图的实现后，下面我们一起看看一条朋友圈信息是如何显示的，首先看一下它的布局图。</p>
<p data-nodeid="1925"><img src="https://s0.lgstatic.com/i/image6/M01/44/1C/Cgp9HWC90huAF_a6AAIy6uuggTs430.png" alt="Drawing 4.png" data-nodeid="2345"></p>
<p data-nodeid="1926">外层是一个<code data-backticks="1" data-nodeid="2347">ZStack</code>，这样能保证<code data-backticks="1" data-nodeid="2349">Toggle</code>可以一直浮动在右下角。<code data-backticks="1" data-nodeid="2351">ZStack</code>还包含一个<code data-backticks="1" data-nodeid="2353">HStack</code>，在<code data-backticks="1" data-nodeid="2355">HStack</code>的左边是一张用于显示朋友头像的图片，右边是一个<code data-backticks="1" data-nodeid="2357">VStack</code>。<code data-backticks="1" data-nodeid="2359">VStack</code>里依次放了显示朋友名字的<code data-backticks="1" data-nodeid="2361">Text</code>、显示标题的<code data-backticks="1" data-nodeid="2363">Text</code>、显示图片的<code data-backticks="1" data-nodeid="2365">KFImage</code>、显示时间的<code data-backticks="1" data-nodeid="2367">Text</code>，以及最底层的<code data-backticks="1" data-nodeid="2369">HStack</code>，这个<code data-backticks="1" data-nodeid="2371">HStack</code>放置了一个心形图片和多个点赞人的头像。其布局代码如下所示， 你可以结合上面的图来理解。</p>
<pre class="lang-swift" data-nodeid="1927"><code data-language="swift"><span class="hljs-type">ZStack</span>(alignment: .bottomTrailing) {
    <span class="hljs-type">HStack</span>(alignment: .top, spacing: <span class="hljs-type">Spacing</span>.medium) {
        <span class="hljs-type">KFImage</span>(viewModel.userAvatarURL)
            .resizable()
            .clipShape(<span class="hljs-type">Circle</span>())
            .frame(width: <span class="hljs-number">44</span>, height: <span class="hljs-number">44</span>)
            .shadow(color: <span class="hljs-type">Color</span>.primary.opacity(<span class="hljs-number">0.15</span>), radius: <span class="hljs-number">5</span>, x: <span class="hljs-number">0</span>, y: <span class="hljs-number">2</span>)
            .padding(.leading, <span class="hljs-type">Spacing</span>.medium)
        <span class="hljs-type">VStack</span>(alignment: .leading) {
            <span class="hljs-type">Text</span>(viewModel.userName)
                .font(.subheadline)
                .foregroundColor(.primary)
            <span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> title = viewModel.title {
                <span class="hljs-type">Text</span>(title)
                    .font(.body)
                    .foregroundColor(<span class="hljs-type">Color</span>.secondary)
            }
            <span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> photoURL = viewModel.photoURL {
                <span class="hljs-type">KFImage</span>(photoURL)
                    .resizable()
                    .frame(width: <span class="hljs-number">240</span>, height: <span class="hljs-number">120</span>)
            }
            <span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> postDateDescription = viewModel.postDateDescription {
                <span class="hljs-type">Text</span>(postDateDescription)
                    .font(.footnote)
                    .foregroundColor(<span class="hljs-type">Color</span>.secondary)
            }
            <span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> likes = viewModel.likes, !likes.isEmpty {
                <span class="hljs-type">HStack</span> {
                    <span class="hljs-type">Image</span>(systemName: <span class="hljs-string">"heart"</span>)
                        .foregroundColor(.secondary)
                    <span class="hljs-type">ForEach</span>(likes.<span class="hljs-built_in">map</span> { <span class="hljs-type">IdentifiableURL</span>(url: $<span class="hljs-number">0</span>) }) {
                        <span class="hljs-type">KFImage</span>($<span class="hljs-number">0</span>.url)
                            .resizable()
                            .frame(width: <span class="hljs-number">20</span>, height: <span class="hljs-number">20</span>)
                            .clipShape(<span class="hljs-type">Circle</span>())
                            .shadow(color: <span class="hljs-type">Color</span>.primary.opacity(<span class="hljs-number">0.15</span>), radius: <span class="hljs-number">3</span>, x: <span class="hljs-number">0</span>, y: <span class="hljs-number">2</span>)
                    }
                }
            }
        }
        <span class="hljs-type">Spacer</span>()
    }
    <span class="hljs-type">Toggle</span>(isOn: $isLiked) {
    }
}
</code></pre>
<p data-nodeid="1928">其中，<code data-backticks="1" data-nodeid="2374">Toggle</code>使用了当前流行的新拟物化设计（Neumorphism），其具有光影效果，同时在点击时会有丝绸物料凸凹变化的效果。那是怎样做到的呢？下面一起看看<code data-backticks="1" data-nodeid="2376">Toggle</code>组件的代码。</p>
<pre class="lang-swift" data-nodeid="1929"><code data-language="swift"><span class="hljs-type">Toggle</span>(isOn: $isLiked) {
    <span class="hljs-type">Image</span>(systemName: <span class="hljs-string">"heart.fill"</span>)
        .foregroundColor(isLiked == <span class="hljs-literal">true</span> ? <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonSelected"</span>) : <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonNotSelected"</span>))
        .animation(.easeIn)
}
.toggleStyle(<span class="hljs-type">LikeToggleStyle</span>())
.padding(.trailing, <span class="hljs-type">Spacing</span>.medium)
.onChange(of: isLiked, perform: { isOn <span class="hljs-keyword">in</span>
    <span class="hljs-keyword">guard</span> isLiked == isOn <span class="hljs-keyword">else</span> { <span class="hljs-keyword">return</span> }
    <span class="hljs-keyword">if</span> isOn {
        viewModel.like(from: userDataStore.currentUser.userID).subscribe().disposed(by: disposeBag)
    } <span class="hljs-keyword">else</span> {
        viewModel.unlike(from: userDataStore.currentUser.userID).subscribe().disposed(by: disposeBag)
    }
})
</code></pre>
<p data-nodeid="1930">我们在<code data-backticks="1" data-nodeid="2379">Toggle</code>里面放了一个心形的<code data-backticks="1" data-nodeid="2381">Image</code>，并根据选中状态来填充不同的颜色。当我们点击<code data-backticks="1" data-nodeid="2383">Toggle</code>时，会根据选中状态来调用<code data-backticks="1" data-nodeid="2385">viewModel</code>的<code data-backticks="1" data-nodeid="2387">like()</code>或者<code data-backticks="1" data-nodeid="2389">unlike()</code>方法，这样就能把选中状态更新到后台去了。</p>
<p data-nodeid="1931">下面看一下如何配置<code data-backticks="1" data-nodeid="2392">Toggle</code>的显示风格。这里我们定义了一个名叫<code data-backticks="1" data-nodeid="2394">LikeToggleStyle</code>的结构体，该结构体遵循了<code data-backticks="1" data-nodeid="2396">ToggleStyle</code>协议。我们可以在<code data-backticks="1" data-nodeid="2398">LikeToggleStyle</code>里面配置<code data-backticks="1" data-nodeid="2400">Toggle</code>的显示风格，代码如下：</p>
<pre class="lang-swift" data-nodeid="1932"><code data-language="swift"><span class="hljs-keyword">private</span> <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">LikeToggleStyle</span>: <span class="hljs-title">ToggleStyle</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">makeBody</span><span class="hljs-params">(configuration: <span class="hljs-keyword">Self</span>.Configuration)</span></span> -&gt; some <span class="hljs-type">View</span> {
        <span class="hljs-type">Button</span>(action: {
            configuration.isOn.toggle()
        }, label: {
            configuration.label
                .padding(<span class="hljs-type">Spacing</span>.extraSmall)
                .contentShape(<span class="hljs-type">Circle</span>())
        })
        .background(
            <span class="hljs-type">LikeToggleBackground</span>(isHighlighted: configuration.isOn, shape: <span class="hljs-type">Circle</span>())
        )
    }
}
</code></pre>
<p data-nodeid="1933">要配置<code data-backticks="1" data-nodeid="2403">Toggle</code>的显示风格，我们需要实现<code data-backticks="1" data-nodeid="2405">makeBody(configuration:)</code>方法来返回一个<code data-backticks="1" data-nodeid="2407">View</code>。在这个<code data-backticks="1" data-nodeid="2409">View</code>里面包含了一个<code data-backticks="1" data-nodeid="2411">Button</code>组件来处理用户的点击事件，当用户点击的时候，我们会改变了<code data-backticks="1" data-nodeid="2413">isOn</code>属性的值。除了按钮以外，我们还使用了<code data-backticks="1" data-nodeid="2415">label</code>参数把<code data-backticks="1" data-nodeid="2417">Toggle</code>配置成圆形，并通过<code data-backticks="1" data-nodeid="2419">background()</code>方法来进行绘制，绘制 UI 的代码都封装在<code data-backticks="1" data-nodeid="2421">LikeToggleBackground</code>里面。下面一起看看它的实现代码：</p>
<pre class="lang-swift" data-nodeid="1934"><code data-language="swift"><span class="hljs-keyword">private</span> <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">LikeToggleBackground</span>&lt;<span class="hljs-title">S</span>: <span class="hljs-title">Shape</span>&gt;: <span class="hljs-title">View</span> </span>{
    <span class="hljs-keyword">var</span> isHighlighted: <span class="hljs-type">Bool</span>
    <span class="hljs-keyword">var</span> shape: <span class="hljs-type">S</span>
    <span class="hljs-keyword">var</span> body: some <span class="hljs-type">View</span> {
        <span class="hljs-type">ZStack</span> {
            <span class="hljs-keyword">if</span> isHighlighted {
                shape
                    .fill(<span class="hljs-type">LinearGradient</span>(<span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonFillEnd"</span>), <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonFillStart"</span>)))
                    .overlay(shape.stroke(<span class="hljs-type">LinearGradient</span>(<span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonFillStart"</span>), <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonFillEnd"</span>)), lineWidth: <span class="hljs-number">2</span>))
                    .shadow(color: <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonStart"</span>), radius: <span class="hljs-number">5</span>, x: <span class="hljs-number">5</span>, y: <span class="hljs-number">5</span>)
                    .shadow(color: <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonEnd"</span>), radius: <span class="hljs-number">5</span>, x: -<span class="hljs-number">5</span>, y: -<span class="hljs-number">5</span>)
            } <span class="hljs-keyword">else</span> {
                shape
                    .fill(<span class="hljs-type">LinearGradient</span>(<span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonFillStart"</span>), <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonFillEnd"</span>)))
                    .overlay(shape.stroke(<span class="hljs-type">LinearGradient</span>(<span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonFillStart"</span>), <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonFillEnd"</span>)), lineWidth: <span class="hljs-number">2</span>))
                    .shadow(color: <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonStart"</span>), radius: <span class="hljs-number">5</span>, x: <span class="hljs-number">5</span>, y: <span class="hljs-number">5</span>)
                    .shadow(color: <span class="hljs-type">Color</span>(<span class="hljs-string">"likeButtonEnd"</span>), radius: <span class="hljs-number">5</span>, x: -<span class="hljs-number">5</span>, y: -<span class="hljs-number">5</span>)
            }
        }
    }
}
</code></pre>
<p data-nodeid="1935">在<code data-backticks="1" data-nodeid="2424">LikeToggleBackground</code>里面，我们根据<code data-backticks="1" data-nodeid="2426">isHighlighted</code>属性的选中状态，为图形填充不同的颜色和阴影效果，从而做出丝绸材质的效果。<br>
最后看看朋友圈信息视图的外层显示风格，代码如下：</p>
<pre class="lang-swift" data-nodeid="1936"><code data-language="swift">.frame(maxWidth:.infinity)
.padding(<span class="hljs-type">EdgeInsets</span>(top: <span class="hljs-type">Spacing</span>.medium, leading: <span class="hljs-number">0</span>, bottom: <span class="hljs-type">Spacing</span>.medium, trailing: <span class="hljs-number">0</span>))
.background(<span class="hljs-type">BlurView</span>(style: .systemMaterial))
.clipShape(<span class="hljs-type">RoundedRectangle</span>(cornerRadius: <span class="hljs-number">30</span>, style: .continuous))
.shadow(color: <span class="hljs-type">Color</span>.black.opacity(<span class="hljs-number">0.15</span>), radius: <span class="hljs-number">20</span>, x: <span class="hljs-number">0</span>, y: <span class="hljs-number">20</span>)
.padding(.horizontal)
</code></pre>
<p data-nodeid="1937">我们调用<code data-backticks="1" data-nodeid="2431">frame(maxWidth:.infinity)</code>和<code data-backticks="1" data-nodeid="2433">padding(.horizontal)</code>方法把<code data-backticks="1" data-nodeid="2435">SwiftUIMomentListItemView</code>的宽度设为设备大小并减去左右两边的留白间距。<code data-backticks="1" data-nodeid="2437">padding(EdgeInsets())</code>方法用于添加上下的间距。通过把自定义的<code data-backticks="1" data-nodeid="2439">BlurView</code>传递给<code data-backticks="1" data-nodeid="2441">background()</code>方法，我们就能实现毛玻璃的显示效果；调用<code data-backticks="1" data-nodeid="2443">clipShape()</code>方法可以来设置大圆角的效果；而调用<code data-backticks="1" data-nodeid="2445">shadow()</code>方法就能完成配置阴影的效果，从而使得朋友圈信息视图有浮动起来的特效。</p>
<p data-nodeid="1938">到此为止，我们已经使用 SwiftUI 实现了整个 View 层了，最后看一下实现的效果，如下动图：</p>
<p data-nodeid="1939"><img src="https://s0.lgstatic.com/i/image6/M00/44/24/CioPOWC90g2ANeBIAAWrFoNi17Q433.png" alt="Drawing 5.png" data-nodeid="2450"></p>
<h3 data-nodeid="1940">总结</h3>
<p data-nodeid="1941">在这一讲，我们介绍了 SwiftUI 管理状态的几种方法，它们之间有些细微的区别，搞清楚它们的工作原理能帮助我们在实践中选择出合适的方法。</p>
<p data-nodeid="1942">另外，我们还讲述了如何使用 SwiftUI 重新实现 Moments App 的 UI 层。你可能已经发现了，在实现的过程中，我们完全没有改动原有的代码，只是在原有代码的基础上进行扩展。一套灵活的框架能帮助我们不断扩展新功能，并无缝引入新技术。</p>
<p data-nodeid="1943">作为开发者，学习新东西已经成为我们生活的一部分。我建议你多花点时间学习一下 SwiftUI，因为现在很多新功能（例如 Widget）只能使用 SwiftUI 进行开发了。后续随着 SwiftUI 的不断成熟，再加上用户设备上 iOS 版本的更新，SwiftUI 慢慢会成为 iOS 乃至苹果所有操作系统开发的主流。</p>
<p data-nodeid="1944"><strong data-nodeid="2458">思考题</strong></p>
<blockquote data-nodeid="1945">
<p data-nodeid="1946">请问你在实际工作中使用过 SwiftUI 吗？能分享一下你的使用经验吗？</p>
</blockquote>
<p data-nodeid="1947">可以把你心得体会写到留言区哦。到此为止，整个课程就学习完毕了，下一讲是结束语，我会把整个课程做一个简单的梳理和串讲，也相当于我们课程的一个小结吧，记住按时来听课哦。</p>
<p data-nodeid="1948"><strong data-nodeid="2464">源码地址</strong></p>
<blockquote data-nodeid="1949">
<p data-nodeid="1950" class="">SwiftUI 实现的 PR：<a href="https://github.com/lagoueduCol/iOS-linyongjian/pull/13?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2468">https://github.com/lagoueduCol/iOS-linyongjian/pull/13</a></p>
</blockquote>

---

### 精选评论

##### **奎：
> 我看了下protocol ObservableObject和extension ObservableObject官方API，objectWillChange好像并没有赋值，我们在使用objectWillChange.send()时候,objectWillChange这个属性是怎么有值的呢？

