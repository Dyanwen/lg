<p>你好，在课程的最开始，我想先来带你从跨多终端技术发展历程看看未来大前端的发展趋势。</p>
<p>随着互联网技术的发展，业务平台对多设备、多终端的需求越来越多，因此，仅掌握单一平台编程开发能力已经稍显欠缺。由于在业务开发过程中，开发者大部分的时间都专研于一种编程语言，如果想要掌握多端开发能力，则又稍显力不从心。因此<strong>大前端</strong>的概念应运而生，本课时主要介绍大前端的发展历程，以及如何做好大前端技术的选型分析。</p>
<h3>什么是大前端</h3>
<p>大前端概念对于编程开发者来说早已耳熟能详，从我的角度来理解这个概念的话，主要是通过同一套编程代码，经过框架编译转化能够适应于多端平台的前端交互界面。当然这里只介绍目前应用较广的三个方面，即 iOS、Andorid 和 Web H5，之后可以再延伸到小程序、TV、Watch 等其他智能设备，如图 1 所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1A/92/CgqCHl7dpsyAGXTdAABOhwpYTPY773.jpg" alt="01-1-1.jpg"></p>
<p>图 1 多端平台的前端交互界面</p>
<p>大前端的核心是为了解决多端不一致和人力的问题。比如在一些交互复杂度不高的应用中，通过这种模式可以更好地节省人力成本，特别是在一些前期快速发展的创业公司，可以使用较少的人力来支撑一些核心业务功能。</p>
<h3>跨端技术的发展</h3>
<p>移动端跨平台技术经过了一个漫长的发展史，如图 2 所示。下面主要介绍下在这个发展过程中跨平台技术有了哪些进步或者做了哪些优化。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1A/87/Ciqc1F7dpuiAej-kAAAoqW57xyE079.jpg" alt="01-2-1.jpg"></p>
<p>图 2 跨端技术的发展过程</p>
<table>
<thead>
<tr>
<th align="left"><strong>技术</strong></th>
<th align="left"><strong>核心</strong></th>
<th align="left"><strong>原生支持</strong></th>
<th align="left"><strong>动态性</strong></th>
<th align="left"><strong>性能</strong></th>
<th align="left"><strong>体验</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">Ionic/Cordova</td>
<td align="left">JSBridge 封装给 Web 调用</td>
<td align="left">90%</td>
<td align="left">中</td>
<td align="left">差</td>
<td align="left">差</td>
</tr>
<tr>
<td align="left">React Native/Weex</td>
<td align="left">JIT 模式应用 JS 与原生通信</td>
<td align="left">20%</td>
<td align="left">好</td>
<td align="left">中</td>
<td align="left">中</td>
</tr>
<tr>
<td align="left">Flutter</td>
<td align="left">自渲染</td>
<td align="left">5%</td>
<td align="left">优</td>
<td align="left">良</td>
<td align="left">优</td>
</tr>
</tbody>
</table>
<ul>
<li>Ionic/Cordova（Hybrid），在技术原理上的核心是，将原生的一些能力通过 JSBridge 封装给 Web 来调用，扩充了 Web 应用能力。但是这种方法有两个不足，一是依赖客户端，二是在性能和体验上都非常依赖于 Web 端。因此，整体上的体验不可预知。目前这个技术还经常被应用到，例如，当前 App 内会提供白名单域名和可调用的 JSBridge 方法，由此来增强 H5 与客户端交互能力，从而提升 App 内 H5 的灵活性。</li>
<li>React Native/Weex，在原来的 Hybrid 的 JSBridge 基础上进行改进，将 JavaScript 的界面以及交互转化为 Native 的控件，从而在体验上和原生界面基本一致。但因为是 JIT 模式，因此需要频繁地在 JavaScript 与 Native 之间进行通信，从而会有一定的性能损耗影响，导致体验上与原生会有一些差异。</li>
<li>Flutter，取长补短，结合了之前的一些优点，解决了与 Native 之间通信的问题，同时也有了自渲染模式（框架自身实现了一套 UI 基础框架，与原来的渲染模式基本一致）。从而在体验和性能上相对之前的两种框架表现都较好，具体是如何做到的，我会在接下来的第 22 课时中详细介绍。</li>
</ul>
<p>经过上面的技术原理和优缺点对比，Flutter 在各方面都表现出了突出的优势，因此把 Flutter 作为入门大前端的核心框架。</p>
<h3>选择 Flutter 的思考</h3>
<p>大前端这个概念从开始到现在已经有整整 10 年时间，那为什么到现在还没有一统江湖呢？其实从我的角度来看，Flutter 也不会创造一统江湖的成果，因为多端或者智能设备的发展终究不会停止，也很难做到统一标准。那为什么我们还要选择 Flutter 来作为大前端核心框架呢？</p>
<p>事实上 Flutter 的确能够为我们解决一些场景问题，<strong>节省人力成本，同时不影响用户体验</strong>。</p>
<p>选择 Flutter 并不是为了代替 iOS 或者 Android，而是做一个技术互补，比如，Flutter 负责业务功能，而 iOS 和 Android 则负责部分的底层交互提供服务给到 Flutter 应用。Flutter 也是在这两年刚刚兴起的，在应用起步初期还需要部分底层的服务与原生平台进行交互。相信再经过一段时间的发展，Flutter 在这方面会不断地优化和提升，也将能够独立覆盖到更多复杂的业务场景。因此希望你能够明白大前端的概念，以及 Flutter 目前的应用场景。</p>
<p>分析完后，下面对目前的技术团队和未来技术团队进行一个简单分析，也可以认为是一个预测。如果你觉得有帮助，那么可以往这方面进行一些尝试，如图 3 所示。大部分开发者会集中在跨端技术团队中；而另一部分核心技术攻坚则在相应的平台技术端（比如 Android 基础技术团队、iOS 基础技术团队或 Web 基础技术团队），为跨端技术团队提供基础技术服务支撑。当然如果跨端技术团队将组件完善并且可通用化，那么跨端技术团队的人员则可以更快地配置组装的方式构建业务功能。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1A/87/Ciqc1F7dpwCAfhZ3AAChalAD2J4693.png" alt="image (3).png"></p>
<p>图 3 技术团队选型</p>
<h3>总结</h3>
<p>本课时首先介绍了大前端的概念以及发展历程，以及从发展历程来看我们选择 Flutter 的原因，其次也分析了未来跨端框架的发展思考，从而得出新大前端团队架构体系。相信通过本课时的学习，你对大前端的概念和发展以及未来的前景有一定的认识，并且在看过本课时以后，应该要为大前端的方向上做一定的技术积累和沉淀。</p>
<p>下一课时，我将讲解 Flutter 框架中使用的编程语言 Dart，为了方便你更好地理解，我将从 JavaScript 角度来讲解。</p>

---

### 精选评论

##### **建：
> <div>跨技术端的3个对比如果能更详细点就好了，虽然知道flutter性能是更好，但是具体的其他底层为什么不够好，看了这个对比，我其实还是不太明白。</div><div>就是“更依赖于 Web 端”这块，不是很明白，可以更具体点吗</div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Hybrid 其实是一个 H5 页面，在每个 APP 中包了一个 H5 的 Web 页面，只是在需要原生功能的地方，通过原生封装一些 JSAPI 给到页面去调用，看起来就像是 H5 拥有了原生 APP 交互功能。所以文章里面说的依赖 Web 的性能就是这个原因。
React Native 以及 Weex ，就改变了用 H5 实现的方式，使用的是原生的界面，但是用户的各类事件操作，都是需要与 JS 进行操作，而 JS 操作后，需要将响应反馈到原生 Native 中，所以需要一个交互过程（ JIT 意思就是运行时编译，就像在运行的时候将 JS 编译为原生界面的过程 ）。你看我这样说，有明白吗？

##### **的巨人：
> 持续打卡，期待自我提升。

##### **远：
> 加油！

##### gjd：
> 没有点出react-native有点遗憾

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; react-native目前我们也在使用，但是更多应用的是热更新，这也是flutter目前没有搞定的一块，react-native在这里是有一定优势。但是性能方面react-native还是差了不少，特别是在一些动画效果上。如果你还需要更多的关于react-native的可以和我再进行交流，可以在github上进行留言，我会回复哈。

##### **的阿科：
> flutter的性能和体验都不错，而且依赖原生少

##### **彬：
> 性能好不好，看开发的能力了

##### **3336：
> Web开发，但不知道原生native 是什么意思

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 原生native一般是指Android或者iOS，使用的语言是Java、Oject-c、Swift

##### *亮：
> 加油

##### *伟：
> flutter的动态性比rn要好？是从哪些方面提现的？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; Flutter 性能会在22课时 和23课时详细讲喔

##### **杰：
> 坐等更新

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 小编欢迎小可爱喔

##### *辉：
> 坚持打卡！

##### *瑞：
> 打卡

