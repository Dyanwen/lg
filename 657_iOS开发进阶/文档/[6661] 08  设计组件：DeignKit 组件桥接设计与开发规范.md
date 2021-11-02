<p data-nodeid="15834" class="">在上一模块“配置与规范”中，我主要介绍了如何统一项目的配置，以及如何制定统一开发和设计规范。</p>
<p data-nodeid="15835">接下来我们将进入基础组件设计模块，我会为你介绍一些在 iOS 开发过程中，工程化实践需要用的组件，比如设计组件、路由组件。除此之外，我还会聊聊在开发中如何支持多语言、动态字体和深色模式等辅助功能，让你的 App 既有国际范，获取更多用户，还能提升用户体验，获得更多好评。</p>
<p data-nodeid="15836">这一讲，我们就先来聊聊公共组件库，以及如何封装基础设计组件。</p>
<h3 data-nodeid="15837">封装公共功能组件库</h3>
<p data-nodeid="15838">随着产品不断发展，我们会发现，越来越多的公共功能可以封装成组件库，从而被各个模块甚至多个 App 共同使用，比如字体、调色板、间距和头像可以封装成 UI 设计组件库，登录会话和权限管理可以封装成登录与鉴权组件库。</p>
<p data-nodeid="15839">通过利用这些公共功能组件库，不仅能节省大量开发时间，不需要我们再为每个模块重复实现类似的功能；还能减少编译时间，因为如果没有独立的组件库，一点代码的改动都会导致整个 App 重新编译与链接。</p>
<p data-nodeid="15840">那么，怎样才能创建和使用公共功能组件库呢？下面我们以一个设计组件库 DesignKit 为例子介绍下具体怎么做。</p>
<h4 data-nodeid="15841">创建内部公共功能组件库</h4>
<p data-nodeid="15842">公共功能组件库根据使用范围可以分为三大类：内部库、私有库和开源库。</p>
<ul data-nodeid="15843">
<li data-nodeid="15844">
<p data-nodeid="15845">内部库是指该库和主项目共享一个 Repo ，它可以共享到主项目的所有模块中。</p>
</li>
<li data-nodeid="15846">
<p data-nodeid="15847">私有库是指该库使用独立的私有 Repo ，它可以共享到公司多个 App 中。</p>
</li>
<li data-nodeid="15848">
<p data-nodeid="15849">开源库是指该库发布到 GitHub 等开源社区提供给其他开发者使用。</p>
</li>
</ul>
<p data-nodeid="15850">这三类库的创建和使用方式都是一致的。<strong data-nodeid="15958">在实际操作中，我们一般先创建内部库，如果今后有必要，可以再升级为私有库乃至开源库</strong>。下面咱们一起看看怎样创建内部库。</p>
<p data-nodeid="15851">为了方便管理各个内部公共功能组件库，首先我们新建一个叫作<strong data-nodeid="15964">Frameworks 的文件夹</strong>来保存所有的内部库。这个文件夹和主项目文件夹（在我们例子中是 Moments）以及 Workplace 文档（Moments.xcworkspace）平衡。例如下面的文件结构：</p>
<pre class="lang-objectivec" data-nodeid="15852"><code data-language="objectivec">Frameworks          Moments             Pods            Moments.xcworkspace
</code></pre>
<p data-nodeid="15853">然后我们通过 CocoaPods 创建和管理这个内部库。</p>
<p data-nodeid="15854">怎么做呢？有两种办法可以完成这项工作，<strong data-nodeid="15974">一种是使用</strong><code data-backticks="1" data-nodeid="15970">pod lib create [pod name]</code>命令。比如在这个案例当中，我们可以在 Frameworks 文件夹下执行<code data-backticks="1" data-nodeid="15972">bundle exec pod lib create DesignKit</code>命令，然后输入邮箱、语言和平台等信息，让 CocoaPods 创建一个 DesignKit.podspec 以及例子项目等一堆文件。具体如下：</p>
<pre class="lang-java" data-nodeid="15855"><code data-language="java">DesignKit         Example           README.md
DesignKit.podspec LICENSE           _Pods.xcodeproj
</code></pre>
<p data-nodeid="15856">DesignKit.podspec 是 DesignKit 库的 Pod 描述文件，用于描述该 Pod 库的一个特定版本信息。它存放在 CocoaPods 的中心 Repo 供使用者查找和使用。</p>
<p data-nodeid="15857">随着这个 Pod 库的迭代，CocoaPods 的中心 Repo 会为每个特定的 Pod 版本存放一个对应的 podspec 文件。每个 podspec 文件都包括 Pod 对应 Repo 的 URL、源码存放的位置、所支持的系统平台及其系统最低版本号、Swift 语言版本，以及 Pod 的名字、版本号和描述等信息。</p>
<p data-nodeid="15858">DesignKit 组件库的 podspec 文件你可以在<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Frameworks/DesignKit/DesignKit.podspec?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="15980">拉勾教育的仓库中</a>找到。下面是该&nbsp;podspec 文件的一些重要配置：</p>
<pre class="lang-java" data-nodeid="15859"><code data-language="java">  s.name             = <span class="hljs-string">'DesignKit'</span>
  s.version          = <span class="hljs-string">'1.0.0'</span>

  s.ios.deployment_target = <span class="hljs-string">'14.0'</span>
  s.swift_versions = <span class="hljs-string">'5.3'</span>
  s.source_files = <span class="hljs-string">'src/**/*'</span>
  s.resources = <span class="hljs-string">'assets/**/*'</span>
</code></pre>
<p data-nodeid="15860"><code data-backticks="1" data-nodeid="15982">name</code>是该组件的名字，<code data-backticks="1" data-nodeid="15984">version</code>是组件的版本号，当我们更新组件的时候同时需要使用 Semantic Versioning（语义化版本号）更新该版本号。</p>
<p data-nodeid="15861"><code data-backticks="1" data-nodeid="15986">ios.deployment_target</code>为该库所支持的平台和所支持平台的最低版本号。<code data-backticks="1" data-nodeid="15988">swift_versions</code>是支持 Swift 语言的版本号。<code data-backticks="1" data-nodeid="15990">source_files</code>是该库的源代码所在的文件夹，在我们例子中是 src。<code data-backticks="1" data-nodeid="15992">resources</code>是该库资源文件所在的文件夹。</p>
<p data-nodeid="15862"><strong data-nodeid="15998">另外一种是手工创建 DesignKit.podspec 文件。我偏向于这一种，因为手工创建出来的项目更简练</strong>。</p>
<p data-nodeid="15863">比如在这里，我们只需要在 Frameworks 新建一个叫作 DesignKit 的文件夹，然后在它下面建立 src 和 assets 这两个文件夹，以及 LICENSE 和 DesignKit.podspec 这两个文件即可。</p>
<p data-nodeid="15864">如下所示：</p>
<pre class="lang-java" data-nodeid="15865"><code data-language="java">DesignKit.podspec LICENSE           assets            src
</code></pre>
<p data-nodeid="15866">以后所有源代码文件都存放在 src 文件夹下面，而图片、Xib 和 Storyboard 等资源文件存放在 assets 文件夹下。</p>
<p data-nodeid="15867">LICENSE 是许可证文件，如果是开源库，我们必须严格选择一个许可证，这样才能方便其他开发者使用我们的库。</p>
<h4 data-nodeid="15868">检测内部公共功能组件库</h4>
<p data-nodeid="15869">为了保证组件库的使用者能顺利安装和使用我们的库，当我们配置好 DesignKit.podspec 文件后，需要执行<code data-backticks="1" data-nodeid="16005">bundle exec pod spec lint</code>命令来检测该 podspec 文件是否正确。如果我们维护的是一个开源库，这一步尤为重要。因为它会影响到使用者的第一印象，因此我们在发布该 Pod 之前需要把每个错误或者警告都修复好。</p>
<p data-nodeid="15870">不过需要注意的是， CocoaPods 对内部库的检测存在一个 Bug， 会显示下面的警告以及错误信息：</p>
<pre data-nodeid="23239" class="te-preview-highlight"><code>WARN | Missing primary key for source attribute 
ERROR | unknown: Encountered an unknown error (Unsupported download strategy `{:path=&gt;"."}`.) during validation
</code></pre>














<p data-nodeid="15873">由于我们创建的是内部库，所以可以忽略这个警告和错误，只要没有其他错误信息就可以了。</p>
<h4 data-nodeid="15874">使用内部公共功能组件库</h4>
<p data-nodeid="15875">使用内部公共功能组件库非常简单，只要在主项目的 Podfile 里面使用<code data-backticks="1" data-nodeid="16011">:path</code>来指定该内部库的路径即可。</p>
<pre class="lang-java" data-nodeid="15876"><code data-language="java">pod <span class="hljs-string">'DesignKit'</span>, :path =&gt; <span class="hljs-string">'./Frameworks/DesignKit'</span>, :inhibit_warnings =&gt; <span class="hljs-keyword">false</span>
</code></pre>
<p data-nodeid="15877">当执行<code data-backticks="1" data-nodeid="16014">bundle exec pod install</code>命令以后，CocoaPods 会在 Pods 项目下建立一个<strong data-nodeid="16024">Development Pods</strong>文件夹来存放所有内部库的相关文件。<br>
<img src="https://s0.lgstatic.com/i/image6/M00/1F/4C/Cgp9HWBRveSAYt47AASIGxbwB9s124.png" alt="Drawing 0.png" data-nodeid="16023"></p>
<p data-nodeid="15878">有了 CocoaPods，我们新建、管理和使用公共组件库就会变得非常简单。下面我们介绍下如何开发设计组件 DesignKit。</p>
<h3 data-nodeid="15879">DesignKit 设计组件</h3>
<p data-nodeid="15880">DesignKit 是一个设计组件，用于封装与 UI 相关的公共组件。为了方便维护，每次新增一个组件，我们最好都建立一个独立的文件夹，例如把 Spacing.swift 放在新建的 Spacing 文件夹中。</p>
<p data-nodeid="15881"><img src="https://s0.lgstatic.com/i/image6/M00/1F/4C/Cgp9HWBRve-AVJZSAACVtoXExgU145.png" alt="Drawing 1.png" data-nodeid="16030"></p>
<p data-nodeid="15882">下面以几乎每个 App 都会使用到的三个组件：间距（Spacing）、头像（Avatar）和点赞按钮（Favorite Button）为例子，介绍下如何封装基础设计组件。</p>
<h4 data-nodeid="15883">间距</h4>
<p data-nodeid="15884">为了呈现信息分组并体现信息的主次关系，所有 App 的所有页面都会使用到间距来添加留白效果。</p>
<p data-nodeid="15885">间距看起来这么简单，为什么我们还需要为其独立封装为一个公共组件呢？主要原因有这么几条。</p>
<ol data-nodeid="15886">
<li data-nodeid="15887">
<p data-nodeid="15888">可以为整个 App 提供一致的体验，因为我们统一定义了所有间距，各个功能模块的 UI 呈现都保持一致。</p>
</li>
<li data-nodeid="15889">
<p data-nodeid="15890">可以减低设计师和开发者的沟通成本，不会再为某些像素值的多与少而争论不休。设计师只使用预先定义的间距，而开发者也只使用在代码中定义好的间距就行了。</p>
</li>
<li data-nodeid="15891">
<p data-nodeid="15892">可以减低设计师的工作量，很多 UI 界面可以只提供一个设计稿来同时支持 iOS、Android 以及移动 Web。因为设计师只提供预先定义的间距名，而不是 hardcoded （硬编码）的像素值。不同设备上像素值有可能不一样，但间距名却能保持一致。</p>
</li>
<li data-nodeid="15893">
<p data-nodeid="15894">在支持响应式设计的时候，这些间距定义可以根据设备的宽度而自动调整。这远比硬编码的像素值灵活很多，例如在 iPhone 中 twoExtraSmall 是 4 points，而在 iPad 中是 6 points。</p>
</li>
</ol>
<p data-nodeid="15895">别看间距公共组件有那么多优点，但实现起来并不难，一个<strong data-nodeid="16044">struct</strong>就搞定了，简直是一本万利的投入。</p>
<pre class="lang-java" data-nodeid="15896"><code data-language="java"><span class="hljs-keyword">public</span> struct Spacing {
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> let twoExtraSmall: CGFloat = <span class="hljs-number">4</span>
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> let extraSmall: CGFloat = <span class="hljs-number">8</span>
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> let small: CGFloat = <span class="hljs-number">12</span>
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> let medium: CGFloat = <span class="hljs-number">18</span>
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> let large: CGFloat = <span class="hljs-number">24</span>
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> let extraLarge: CGFloat = <span class="hljs-number">32</span>
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> let twoExtraLarge: CGFloat = <span class="hljs-number">40</span>
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> let threeExtraLarge: CGFloat = <span class="hljs-number">48</span>
}
</code></pre>
<p data-nodeid="15897">有了上述的定义以后，使用这些间距变得很简单。请看：</p>
<pre class="lang-java" data-nodeid="15898"><code data-language="java"><span class="hljs-keyword">import</span> DesignKit

<span class="hljs-keyword">private</span> let likesStakeView: UIStackView = configure(.init()) {
    $<span class="hljs-number">0</span>.spacing = Spacing.twoExtraSmall
    $<span class="hljs-number">0</span>.directionalLayoutMargins = NSDirectionalEdgeInsets(top: Spacing.twoExtraSmall, leading: Spacing.twoExtraSmall, bottom: Spacing.twoExtraSmall, trailing: Spacing.twoExtraSmall)
}
</code></pre>
<p data-nodeid="15899">我们可以先 import (引入) DesignKit 库，然后通过<code data-backticks="1" data-nodeid="16047">Spacing</code>结构体直接访问预定义的间距，例如<code data-backticks="1" data-nodeid="16049">Spacing.twoExtraSmall</code>。</p>
<h4 data-nodeid="15900">头像组件</h4>
<p data-nodeid="15901">iOS 开发者都知道，头像组件应用广泛，例如在房产 App 中显示中介的头像，在我们例子 Moments App 中显示自己和好友头像，在短视频 App 中显示视频博主头像等。</p>
<p data-nodeid="15902">也许你会问，头像那么简单，为什么需要独立封装为一个组件？原因主要是方便以后改变其 UI 的呈现方式，例如从圆角方形改成圆形，添加边界线（border），添加阴影效果（shadow）等。有了独立的组件以后，我们只需要修改一个地方就能把这个 App 的所有头像一次性地修改呈现效果。</p>
<p data-nodeid="15903">下面是头像组件的实现方式：</p>
<pre class="lang-java" data-nodeid="15904"><code data-language="java"><span class="hljs-keyword">public</span> extension UIImageView {
    <span class="hljs-function">func <span class="hljs-title">asAvatar</span><span class="hljs-params">(cornerRadius: CGFloat = <span class="hljs-number">4</span>)</span> </span>{
        clipsToBounds = <span class="hljs-keyword">true</span>
        layer.cornerRadius = cornerRadius
    }
}
</code></pre>
<p data-nodeid="15905">我们为 UIKit 所提供的<code data-backticks="1" data-nodeid="16056">UIImageView</code>实现了一个扩展方法<code data-backticks="1" data-nodeid="16058">asAvatar(cornerRadius:)</code>，该方法接收<code data-backticks="1" data-nodeid="16060">cornerRadius</code>作为参数来配置圆角的角度，默认值是<code data-backticks="1" data-nodeid="16062">4</code>。</p>
<p data-nodeid="15906">使用也是非常简单，只有创建一个<code data-backticks="1" data-nodeid="16065">UIImageView</code>的实例，然后调用<code data-backticks="1" data-nodeid="16067">asAvatar(cornerRadius:)</code>方法即可。</p>
<pre class="lang-java" data-nodeid="15907"><code data-language="java">    <span class="hljs-keyword">private</span> let userAvatarImageView: UIImageView = configure(.init()) {
        $<span class="hljs-number">0</span>.asAvatar(cornerRadius: <span class="hljs-number">4</span>)
    }
</code></pre>
<p data-nodeid="15908">这是人像组件的显示效果，可以在内部菜单查看。<br>
<img src="https://s0.lgstatic.com/i/image6/M00/1F/49/CioPOWBRvgqAL1THABqNofQMQ_4461.png" alt="Drawing 2.png" data-nodeid="16073"></p>
<h4 data-nodeid="15909">点赞按钮</h4>
<p data-nodeid="15910">可以说，每个具有社交属性的 App 都会用到点赞功能，所以在开发当中，点赞按钮也是必不可少的功能组件。</p>
<p data-nodeid="15911">那么，点赞按钮该如何封装呢？和人像组件十分类似，我们可以通过扩展<code data-backticks="1" data-nodeid="16077">UIButton</code>来实现。示例代码如下：</p>
<pre class="lang-java" data-nodeid="15912"><code data-language="java"><span class="hljs-keyword">public</span> extension UIButton {
    <span class="hljs-function">func <span class="hljs-title">asStarFavoriteButton</span><span class="hljs-params">(pointSize: CGFloat = <span class="hljs-number">18</span>, weight: UIImage.SymbolWeight = .semibold, scale: UIImage.SymbolScale = .<span class="hljs-keyword">default</span>, fillColor: UIColor = UIColor(hex: <span class="hljs-number">0xf1c40f</span>)</span>) </span>{
        let symbolConfiguration = UIImage.SymbolConfiguration(pointSize: pointSize, weight: weight, scale: scale)
        let starImage = UIImage(systemName: <span class="hljs-string">"star"</span>, withConfiguration: symbolConfiguration)
        setImage(starImage, <span class="hljs-keyword">for</span>: .normal)
        let starFillImage = UIImage(systemName: <span class="hljs-string">"star.fill"</span>, withConfiguration: symbolConfiguration)
        setImage(starFillImage, <span class="hljs-keyword">for</span>: .selected)
        tintColor = <span class="hljs-function">fillColor
        <span class="hljs-title">addTarget</span><span class="hljs-params">(self, action: #selector(touchUpInside)</span>, <span class="hljs-keyword">for</span>: .touchUpInside)
    }
}
<span class="hljs-keyword">private</span> extension UIButton </span>{
    <span class="hljs-meta">@objc</span>
    <span class="hljs-function"><span class="hljs-keyword">private</span> func <span class="hljs-title">touchUpInside</span><span class="hljs-params">(sender: UIButton)</span> </span>{
        isSelected = !isSelected
    }
}
</code></pre>
<p data-nodeid="15913">其核心逻辑把当前 UIButton 对象的普通 (<code data-backticks="1" data-nodeid="16080">.normal</code>) 状态和选中 (<code data-backticks="1" data-nodeid="16082">.selected</code>) 状态设置不同的图标。比如在这里我就把星星按钮的普通状态设置成了名叫 “Star” 的图标，并把它的选中状态设置成了名叫 “tar.fill"” 的图标。</p>
<p data-nodeid="15914">注意，这些图标来自苹果公司的 SF Symbols 不需要额外安装，iOS 14 系统本身就自带了。而且它们的使用也非常灵活，支持字号、字重、填充色等配置。</p>
<p data-nodeid="15915">使用点赞按钮组件也非常简单，只需要建立一个<code data-backticks="1" data-nodeid="16088">UIButton</code>的实例，然后调用<code data-backticks="1" data-nodeid="16090">asStarFavoriteButton</code>方法就可以了。</p>
<pre class="lang-java" data-nodeid="15916"><code data-language="java">    <span class="hljs-keyword">private</span> let favoriteButton: UIButton = configure(.init()) {
        $<span class="hljs-number">0</span>.asStarFavoriteButton()
    }
</code></pre>
<p data-nodeid="15917">点赞按钮的运行效果，也可以在内部菜单查看。</p>
<p data-nodeid="15918">以上我们以间距、头像、点赞按钮为例介绍了如何使用 DesignKit 封装与 UI 相关的公共组件。以我多年的开发经验来说，在封装 UI 组件的时候，可以遵循下面几个原则。</p>
<ol data-nodeid="15919">
<li data-nodeid="15920">
<p data-nodeid="15921">尽量使用扩展方法而不是子类来扩展组件，这样做可以使其他开发者在使用这些组件时，仅需要调用扩展方法，而不必使用特定的类。</p>
</li>
<li data-nodeid="15922">
<p data-nodeid="15923">尽量使用代码而不要使用 Xib 或者 Storyboard，因为有些 App 完全不使用 Interface Builder。</p>
</li>
<li data-nodeid="15924">
<p data-nodeid="15925">如果可以，要为组件加上<code data-backticks="1" data-nodeid="16097">@IBDesignable</code>和<code data-backticks="1" data-nodeid="16099">@IBInspectable</code>支持，这样能使得开发者在使用 Interface Builder 的时候预览我们的组件。</p>
</li>
<li data-nodeid="15926">
<p data-nodeid="15927">尽量只使用 UIkit 而不要依赖任何第三方库，否则我们可能会引入一个不可控的依赖库。</p>
</li>
</ol>
<h3 data-nodeid="15928">总结</h3>
<p data-nodeid="15929">前面我介绍了如何封装公共功能组件库，以及以怎样封装基础设计组件，希望对你有所帮助。合理使用功能组件可以让你的开发事半功倍。<br>
<img src="https://s0.lgstatic.com/i/image6/M00/1F/73/Cgp9HWBR3wyAAnwjAAcCv2pASBs854.png" alt="思维导图+二维码.png" data-nodeid="16107"><br>
不过，在封装组件的时候，我还需要提醒你注意这么几点。</p>
<p data-nodeid="15930">首先，为了减低组件之间的耦合性，提高组件的健壮性，组件的设计需要符合单一功能原则 。也就是说，一个组件只做一件事情，一个组件库只做一类相关的事情。每个组件库都要相对独立且功能单一。</p>
<p data-nodeid="15931">比如，我们可以分别封装网络库、UI 库、蓝牙处理库等底层库，但不能把所有库合并在一个单独的库里面，这样可以方便上层应用按需使用这些依赖库。例如，广告 SDK 可以依赖于网络库、UI 库，但并不依赖蓝牙处理库。这样做一方面可以减少循环依赖的可能性，另一方面可以加快编译和链接的速度，方便使用。</p>
<p data-nodeid="15932">其次，每次发布新增和更新组件的时候，都需要严格按照 Semantic Versioning 来更新版本号，这样有效防止因为版本的问题而引入 Bug。</p>
<p data-nodeid="15933">最后，组件的开发并不是一蹴而就，很多时候可以根据业务需求把公共模块一点点地移入公共组件库中，一步步地完善组件库的功能。不要为了开发组件而开发组件，很多时候当我们充分理解了使用者的需求后，才能为组件定义完善的接口和完整的功能。</p>
<p data-nodeid="15934">思考题：</p>
<blockquote data-nodeid="15935">
<p data-nodeid="15936">上面我们讲述了如何使用 CocoaPods 来封装内部组件，请问怎样把内部组件升级成为私有组件和开源组件呢？</p>
</blockquote>
<p data-nodeid="15937">可以把回答写到下面的留言区哦，我们下一讲将介绍如何使用功能开关支持产品快速迭代。</p>
<p data-nodeid="15938">源码地址：</p>
<blockquote data-nodeid="15939">
<p data-nodeid="15940" class="">DesignKit 源代码：<a href="https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Frameworks/DesignKit?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="16121">https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Frameworks/DesignKit</a></p>
</blockquote>

---

### 精选评论

##### **7867：
> - ERROR | [iOS] unknown: Encountered an unknown error (Unsupported download strategy `{:path="."}`.) during validation.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦，谢谢反馈这个问题，我重新查了一下，发现这是一个 CocoaPods 已知的问题，我目前还没有找到解决方案，可以参考 https://github.com/CocoaPods/guides.cocoapods.org/issues/140，我把原文也更新了。

##### *虎：
> 私有组件：1.创建远程">PrivatePodsgit 仓库2.本地添加私有仓库: pod repo addPrivatePods远程地址3.在私有组件的 Podspec 里配置 s.source = 组件源码仓库地址4.发布私有组件 （tagPrivatePods 私有组件.podspec）4.在主项目的 Podfile 里配置 source =PrivatePods 远程仓库地址6. pod “私有组件名称” （~ tag)7. pod install开源组件：相比私有组件而言，只是把私有组件的 .podspec 文件配置到 cocoapods 的 Github 仓库上

##### **超：
> 林大，如果两个组件相互依赖怎么办？比如DesignKit要用到DataStoreKit里面的一个方法。我在开发DesignKit的时候又想采用文中bundle exec pod lib create DesignKit的方式，这种形势下我在DesignKit-Example项目中怎么引用DataStoreKit？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果两个组件直接相互依赖，那说明程序架构出问题了，相互耦合。应该把共享部分单独出一个库，让两个库分别依赖新的库。在新的库中通过协议来共享给两个库使用，你可以参考一下路由组件模块的  Navigating 协议。有了它，其他模块都可以解耦了，在不相互依赖的情况下进行导航。

##### *豪：
> pod package这个会讲嘛？最近打包动态库一直失败

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个不会特别讲了，请问是错什么错呢？有 Google 出类似的问题吗？

##### **临：
> 干货满满的，就是更新有些慢了

##### **用户5057：
> 能讲一下怎么做成 framework这种形式

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是一个思考题呀，“怎样把内部组件升级成为私有组件和开源组件？”，你可以参考一下我的一个开源项目来试试，这是该项目的 Podspec 文件 https://github.com/IBAnimatable/IBAnimatable/blob/master/IBAnimatable.podspec

##### **泽：
> 文中说的手工创建 DesignKit.podspec 文件 以及 src assert文件夹。那组件库源码是在另一个xcode工程中创建编写测试完， 再拷贝到src文件夹中，是这样一个流程吗

##### **冠：
> 您好，我们的项目也有类似的UI组件库，但当组件库的组件越来越多的时候，即便是很熟悉这个项目的工程师也会忘记有过这个组件，新来的工程师更是不知道都封装过什么，导致有些时候大家会写重复的代码却浑然不知。我觉得这件事靠自己记下来或是说让新加入的工程师自己去熟悉代码成本都比较高，想请教一下老师您有没有什么好的建议？感谢~

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个问题还好呀，我们通常通过一个文档网站来记录所有的组件，例如 https://backpack.github.io/components/badge?platform=ios 。这样大家在实现组建的时候先去查一下，好的网站还包含了各个平台的实例代码，方便大家在不同平台实现同一个功能。

##### *杨：
> 有一个问题, 一个repo pod只干一件事我很赞同, 但是多repo下有什么好的提交代码工具或者方式吗?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你说的对，多个 Pod/Repo 会增加维护成本，所以我一般都建立内部 Pod，就像例子中的 DesignKit 那样。如果是多个 Pod，开发 Pod 和开发主 App 的功能应该不在同一时间进行，否则可能有 code smell。Pod 的开发一般都是公共组件，一旦完成很少修改，假如要修改也不应该在开发功能的时候修改。

##### **一：
> 对于Spacing，使用 enum 会是一个更好的解决方案。struct 仍然存在一个默认 init 方法，而 enum 没有 case 是不存在默认 init 方法的。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 嗯嗯，我是为了演示如何使用 struct 和 enum 两种不同的方案来做命名空间。

##### **泽：
> 我也在执行 bundle exec pod spec lint 遇到这个问题- ERROR | [iOS] unknown: Encountered an unknown error (Unsupported download strategy `{:path="."}`.) during validation.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦，谢谢反馈这个问题，我重新查了一下，发现这是一个 CocoaPods 已知的问题，我目前还没有找到解决方案，可以参考 https://github.com/CocoaPods/guides.cocoapods.org/issues/140，我把原文也更新了。

##### *浩：
> @discardableResultfunc configure T { closure(object) return object}这个方法能详细讲一下吗 并且用这个configure 实现init的优点是什么

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个方法在 21 讲有讲述，有了该方法，我们就可以把所有初始化操作都放在一个闭包（Closure）里面，方便代码的维护。

##### **泽：
> 老师， 通过第二种方式，是自己按照podspec模版来写吗？然后再别的demo工程里把写好的源文件放到src文件夹下吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问第二种方式是那种方式呀？能具体一点吗？如果是内部 Pod 的方式，只要把代码放到 src 文件夹下面即可。

##### **泽：
> 老师，pod私有库的时候，遇到静态库的依赖问题一般怎么解决？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问静态库的依赖的什么问题，能具体一点吗？谢谢。

##### **泽：
> 私有库就是打包成framework或.a的形式提供给别人使用。公共库的话做成pod放到开源网站如github上边。 是这样的吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以的，它们和内部库不一样的地方是和主项目不在一个 Repo 下面，管理成本高一些。

##### **6383：
> Unsupported download strategy `{:path="."}`.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦，谢谢反馈这个问题，我重新查了一下，发现这是一个 CocoaPods 已知的问题，我目前还没有找到解决方案，可以参考 https://github.com/CocoaPods/guides.cocoapods.org/issues/140，我把原文也更新了。

##### **军：
> 老师有空讲下swift module这块知识

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问你想的 Module 是指什么？一个能重用的模块吗，还是一个 Framework ， 可以是文章里面讲述的一个 Pod 吗？

##### *朋：
> 大佬对于 使用 target 管理 framework 是什么看法？ 把基础库，网络库封装到不同的 target的 framework中。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Target 也是可以的，假如你不想做成 Pod，可以使用 Target 来管理 Framework。因为生成维护一个 Pod 的成本不高，我推荐做成 Pod ，以后就可以升级成私有共享库或者开源库。

##### **龙：
> 感觉使用Swift Package进行本地组件化会更为简单，目前我会考虑使用SP，如果需要作为私有库甚至公有库的话，会Swift Package和Cocopods管理合一，现在很多主流的框架也是这么干的。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为 Swift Package Manager 还是有一些限制，例如对 Objective-C 支持不好，二进制库支持不好等。但是以后都会慢慢转到 Swift Package Manager。我在文章中讲述如何使用 CocoaPods 管理本地组件的维护成本其实很低呀，只需要搭建一次，以后可以不断添加源码文件和资源文件。

##### **2047：
> 间距单词有点长，可以改成，xsmall,xlarge，xxlarge

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以的，只要公司统一，开发者和设计师都同意就行，规范的目的是方便沟通。

##### **军：
> 请教下老师，swift私有库封装微信分享时，因为有桥接文件，pod lib spec 验证不通过怎么解决？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 微信分享是一个公开 Pod 吗？具体的问题我也没有碰到过，有错误信息吗？能不能 Google 一下看看有没有其他人碰到过并解决了？

##### jake：
> 老师， 对比用cocoapod来创建管理internal library, 我直接在项目里面创建一个新的target framework有什么优缺点？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用 CocoaPods 来管理，以后可以慢慢地升级为其他库，例如私有组件库和开源组件库等等。

##### *鹏：
> 封装内部组件，使用pod spec lint分析时，出现了ERROR | [iOS] unknown: Encountered an unknown error (Unsupported download strategy `{:path="."}`.) during validation错误，是什么原因呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦，谢谢反馈这个问题，我重新查了一下，发现这是一个 CocoaPods 已知的问题，我目前还没有找到解决方案，可以参考 https://github.com/CocoaPods/guides.cocoapods.org/issues/140，我把原文也更新了。

