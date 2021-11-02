<p data-nodeid="2019" class="">Android touch 事件的分发是 Android 工程师必备技能之一。关于事件分发主要有几个方向可以展开深入分析：</p>
<ol data-nodeid="3717">
<li data-nodeid="3718">
<p data-nodeid="3719">touch 事件是如何从驱动层传递给 Framework 层的 InputManagerService；</p>
</li>
<li data-nodeid="3720">
<p data-nodeid="3721" class="te-preview-highlight">WMS 是如何通过 ViewRootImpl 将事件传递到目标窗口；</p>
</li>
<li data-nodeid="3722">
<p data-nodeid="3723">touch 事件到达 DecorView 后，是如何一步步传递到内部的子 View 中的。</p>
</li>
</ol>




<p data-nodeid="2027">其中与上层软件开发息息相关的就是第 3 条，也是本课时的重点关注内容。</p>
<blockquote data-nodeid="2028">
<p data-nodeid="2029">注意：不同版本间的代码会有区别，本文是基于 Android-28 的源码上进行分析。</p>
</blockquote>
<h3 data-nodeid="2030">思路梳理</h3>
<p data-nodeid="2031">在深入分析事件分发源码之前，需要先弄清楚 2 个概念。</p>
<h4 data-nodeid="2032">ViewGroup</h4>
<p data-nodeid="2033">ViewGroup 是一组 View 的组合，在其内部有可能包含多个子 View，当手指触摸屏幕上时，手指所在的区域既能在 ViewGroup 显示范围内，也可能在其内部 View 控件上。</p>
<p data-nodeid="2034">因此它内部的事件分发的重心是处理当前 Group 和子 View 之间的逻辑关系：</p>
<ol data-nodeid="2035">
<li data-nodeid="2036">
<p data-nodeid="2037">当前 Group 是否需要拦截 touch 事件；</p>
</li>
<li data-nodeid="2038">
<p data-nodeid="2039">是否需要将 touch 事件继续分发给子 View；</p>
</li>
<li data-nodeid="2040">
<p data-nodeid="2041">如何将 touch 事件分发给子 View。</p>
</li>
</ol>
<h4 data-nodeid="2042">View</h4>
<p data-nodeid="2043">View 是一个单纯的控件，不能再被细分，内部也并不会存在子 View，所以它的事件分发的重点在于当前 View 如何去处理 touch 事件，并根据相应的手势逻辑进行一些列的效果展示（比如滑动，放大，点击，长按等）。</p>
<ol data-nodeid="2044">
<li data-nodeid="2045">
<p data-nodeid="2046">是否存在 TouchListener；</p>
</li>
<li data-nodeid="2047">
<p data-nodeid="2048">是否自己接收处理 touch 事件（主要逻辑在 onTouchEvent 方法中）。</p>
</li>
</ol>
<h3 data-nodeid="2049">事件分发核心 dispatchTouchEvent</h3>
<p data-nodeid="2050">整个 View 之间的事件分发，实质上就是一个大的递归函数，而这个递归函数就是 dispatchTouchEvent 方法。在这个递归的过程中会适时调用 onInterceptTouchEvent 来拦截事件，或者调用 onTouchEvent 方法来处理事件。</p>
<p data-nodeid="2051">先从宏观角度，纵览整个 dispatch 的源码如下：</p>
<p data-nodeid="2052"><img src="https://s0.lgstatic.com/i/image/M00/04/0D/Ciqc1F6zq76AYtJPAADS4BOo8VM039.png" alt="image.png" data-nodeid="2167"></p>
<p data-nodeid="2053">如代码中的注释，dispatch 主要分为 3 大步骤：</p>
<ul data-nodeid="2054">
<li data-nodeid="2055">
<p data-nodeid="2056">步骤 1：判断当前 ViewGroup 是否需要拦截此 touch 事件，如果拦截则此次 touch 事件不再会传递给子 View（<strong data-nodeid="2174">或者以 CANCEL 的方式通知子 View</strong>）。</p>
</li>
<li data-nodeid="2057">
<p data-nodeid="2058">步骤 2：如果没有拦截，则将事件分发给子 View 继续处理，如果子 View 将此次事件捕获，则将 mFirstTouchTarget 赋值给捕获 touch 事件的 View。</p>
</li>
<li data-nodeid="2059">
<p data-nodeid="2060">步骤 3：根据 mFirstTouchTarget 重新分发事件。</p>
</li>
<li data-nodeid="2061">
<p data-nodeid="2062">接下来详细的看下每一个步骤：</p>
</li>
</ul>
<h4 data-nodeid="2063">步骤 1 的具体代码如下</h4>
<p data-nodeid="2064"><img src="https://s0.lgstatic.com/i/image/M00/04/0D/Ciqc1F6zq86AZd-ZAAHnUYfq8iQ886.png" alt="image (1).png" data-nodeid="2181"></p>
<p data-nodeid="2065">图中红框标出了是否需要拦截的条件：</p>
<ul data-nodeid="2066">
<li data-nodeid="2067">
<p data-nodeid="2068">如果事件为 DOWN 事件，则调用 onInterceptTouchEvent 进行拦截判断；</p>
</li>
<li data-nodeid="2069">
<p data-nodeid="2070">或者 mFirstTouchTarget 不为 null，代表已经有子 View 捕获了这个事件，子 View 的 dispatchTouchEvent 返回 true 就是代表捕获 touch 事件。</p>
</li>
</ul>
<p data-nodeid="2071">如果在上面步骤 1 中，当前 ViewGroup 并没有对事件进行拦截，则执行步骤 2。</p>
<h4 data-nodeid="2072">步骤 2 具体代码如下</h4>
<p data-nodeid="2073"><img src="https://s0.lgstatic.com/i/image/M00/04/0D/Ciqc1F6zq9mAMTcTAAdFapIjIMc616.png" alt="image (2).png" data-nodeid="2189"></p>
<p data-nodeid="2074">仔细看上述的代码可以看出：</p>
<ul data-nodeid="2075">
<li data-nodeid="2076">
<p data-nodeid="2077">图中 ① 处表明事件主动分发的前提是事件为 DOWN 事件；</p>
</li>
<li data-nodeid="2078">
<p data-nodeid="2079">图中 ② 处遍历所有子 View；</p>
</li>
<li data-nodeid="2080">
<p data-nodeid="2081">图中 ③ 处判断事件坐标是否在子 View 坐标范围内，并且子 View 并没有处在动画状态；</p>
</li>
<li data-nodeid="2082">
<p data-nodeid="2083">图中 ④ 处调用 dispatchTransformedTouchEvent 方法将事件分发给子 View，如果子 View 捕获事件成功，则将 mFirstTouchTarget 赋值给子 View。</p>
</li>
</ul>
<h4 data-nodeid="2084">步骤 3 具体代码如下</h4>
<p data-nodeid="2085"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/Ciqc1F6zq-OAV7ZcAAbboCsB_8o465.png" alt="image (3).png" data-nodeid="2198"></p>
<p data-nodeid="2086">步骤 3 有 2 个分支判断。</p>
<ul data-nodeid="2087">
<li data-nodeid="2088">
<p data-nodeid="2089">分支 1：如果此时 mFirstTouchTarget 为 null，说明在上述的事件分发中并没有子 View 对事件进行了捕获操作。这种情况下，直接调用 dispatchTransformedTouchEvent 方法，并传入 child 为 null，最终会调用 super.dispatchTouchEvent 方法。实际上最终会调用自身的 onTouchEvent 方法，进行处理 touch 事件。也就是说：<strong data-nodeid="2204">如果没有子 View 捕获处理 touch 事件，ViewGroup 会通过自身的 onTouchEvent 方法进行处理。</strong></p>
</li>
<li data-nodeid="2090">
<p data-nodeid="2091">分支 2：mFirstTouchTarget 不为 null，说明在上面步骤 2 中有子 View 对 touch 事件进行了捕获，则直接将当前以及后续的事件交给 mFirstTouchTarget 指向的 View 进行处理。</p>
</li>
</ul>
<h4 data-nodeid="2092">事件分发流程代码演示</h4>
<p data-nodeid="2093">定义如下布局文件：</p>
<p data-nodeid="2094"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/Ciqc1F6zq_GANeaVAAKS6a5wrDA923.png" alt="image (4).png" data-nodeid="2210"></p>
<p data-nodeid="2095">DownInterceptedGroup 和 CaptureTouchView 是两个自定义 View，它们的源码分别如下：</p>
<p data-nodeid="2096"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/CgqCHl6zq_mAFXqUAAVCCMHCxbY587.png" alt="image (5).png" data-nodeid="2214"></p>
<p data-nodeid="2097">手指触摸 CaptureTouchView 并滑动一段距离后抬起，最终打印 log 如下：</p>
<p data-nodeid="2098"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/CgqCHl6zrAGAWq9uAAWUXplsW3M487.png" alt="image (6).png" data-nodeid="2218"></p>
<p data-nodeid="2099">上图中在 DOWN 事件中，DownInterceptGroup 的 onInterceptTouchEvent 被触发一次；然后在子 View CaptureTouchView 的 dispatchTouchEvent 中返回 true，代表它捕获消费了这个 DOWN 事件。这种情况下 CaptureTouchView 会被添加到父视图（DownInterceptGroup）中的 mFirstTouchTarget 中。因此后续的 MOVE 和 UP 事件都会经过 DownInterceptGroup 的 onInterceptTouchEvent 进行拦截判断。 详细源码可以参考：<a href="https://github.com/McoyJiang/LagouAndroidShare/blob/master/course14_%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91/LagouTouchExplanation/app/src/main/java/material/danny_jiang/com/lagoutouchexplanation/views/CaptureTouchView.java" data-nodeid="2222">CaptureTouchView.java</a></p>
<h3 data-nodeid="2100">为什么 DOWN 事件特殊</h3>
<p data-nodeid="2101">所有 touch 事件都是从 DOWN 事件开始的，这是 DOWN 事件比较特殊的原因之一。另一个原因是 DOWN 事件的处理结果会直接影响后续 MOVE、UP 事件的逻辑。</p>
<p data-nodeid="2102">在步骤 2 中，只有 DOWN 事件会传递给子 View 进行捕获判断，一旦子 View 捕获成功，后续的 MOVE 和 UP 事件是通过遍历 mFirstTouchTarget 链表，查找之前接受 ACTION_DOWN 的子 View，并将触摸事件分配给这些子 View。<strong data-nodeid="2231">也就是说后续的 MOVE、UP 等事件的分发交给谁，取决于它们的起始事件 Down 是由谁捕获的。</strong></p>
<h3 data-nodeid="2103">mFirstTouchTarget 有什么作用</h3>
<p data-nodeid="2104">mFirstTouchTarget 的部分源码如下：</p>
<p data-nodeid="2105"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/Ciqc1F6zrAqAHqzwAAFLLP0HYng235.png" alt="image (7).png" data-nodeid="2236"></p>
<p data-nodeid="2106">可以看出其实 mFirstTouchTarget 是一个 TouchTarget 类型的<strong data-nodeid="2242">链表</strong>结构。而这个 TouchTarget 的作用就是用来记录捕获了 DOWN 事件的 View，具体保存在上图中的 child 变量。可是为什么是链表类型的结构呢？因为 Android 设备是支持多指操作的，每一个手指的 DOWN 事件都可以当做一个 TouchTarget 保存起来。在步骤 3 中判断如果 mFirstTouchTarget 不为 null，则再次将事件分发给相应的 TouchTarget。</p>
<h3 data-nodeid="2107">容易被遗漏的 CANCEL 事件</h3>
<p data-nodeid="2108">在上面的步骤 3 中，继续向子 View 分发事件的代码中，有一段比较有趣的逻辑：</p>
<p data-nodeid="2109"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/Ciqc1F6zrBKACMw7AAINKPG9H7g994.png" alt="image (8).png" data-nodeid="2247"></p>
<p data-nodeid="2110">上图红框中表明已经有子 View 捕获了 touch 事件，但是蓝色框中的 intercepted boolean 变量又是 true。这种情况下，事件主导权会重新回到父视图 ViewGroup 中，并传递给子 View 的分发事件中传入一个 cancelChild == true。</p>
<p data-nodeid="2111">看一下 dispatchTransformedTouchEvent 方法的部分源码如下：</p>
<p data-nodeid="2112"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/CgqCHl6zrBqAQOmTAAGn_cFIxaU530.png" alt="image (9).png" data-nodeid="2252"></p>
<p data-nodeid="2113">因为之前传入参数 cancel 为 true，并且 child 不为 null，<strong data-nodeid="2260">最终这个事件会被包装为一个 ACTION_CANCEL 事件传给 child</strong>。</p>
<p data-nodeid="2114"><strong data-nodeid="2264">什么情况下会触发这段逻辑呢？</strong></p>
<blockquote data-nodeid="2115">
<p data-nodeid="2116">总结一下就是：当父视图的 onInterceptTouchEvent 先返回 false，然后在子 View 的 dispatchTouchEvent 中返回 true（表示子 View 捕获事件），关键步骤就是在接下来的 MOVE 的过程中，父视图的 onInterceptTouchEvent 又返回 true，intercepted 被重新置为 true，此时上述逻辑就会被触发，子控件就会收到 ACTION_CANCEL 的 touch 事件。</p>
</blockquote>
<p data-nodeid="2117"><strong data-nodeid="2273">实际上有个很经典的例子可以用来演示这种情况：</strong><br>
当在 Scrollview 中添加自定义 View 时，ScrollView 默认在 DOWN 事件中并不会进行拦截，事件会被传递给 ScrollView 内的子控件。只有当手指进行滑动并到达一定的距离之后，onInterceptTouchEvent 方法返回 true，并触发 ScrollView 的滚动效果。当 ScrollView 进行滚动的瞬间，内部的子 View 会接收到一个 CANCEL 事件，并丢失touch焦点。</p>
<p data-nodeid="2118">比如以下代码：</p>
<p data-nodeid="2119"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/Ciqc1F6zrCKAVemaAAQcKt0yEQc281.png" alt="image (10).png" data-nodeid="2277"></p>
<p data-nodeid="2120">CaptureTouchView 是一个自定义的 View，其源码如下：</p>
<p data-nodeid="2121"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/CgqCHl6zrCqAU5jRAAHMKJUl2y4204.png" alt="image (11).png" data-nodeid="2281"></p>
<p data-nodeid="2122">CaptureTouchView 的 onTouchEvent 返回 true，表示它会将接收到的 touch 事件进行捕获消费。</p>
<p data-nodeid="2123">上述代码执行后，当手指点击屏幕时 DOWN 事件会被传递给 CaptureTouchView，手指滑动屏幕将 ScrollView 上下滚动，刚开始 MOVE 事件还是由 CaptureTouchView 来消费处理，但是当 ScrollView 开始滚动时，CaptureTouchView 会接收一个 CANCEL 事件，并不再接收后续的 touch 事件。具体打印 log 如下：</p>
<p data-nodeid="2124"><img src="https://s0.lgstatic.com/i/image/M00/04/0E/CgqCHl6zrDGAW0S9AAKvCBLfnyM142.png" alt="image (12).png" data-nodeid="2286"></p>
<blockquote data-nodeid="2125">
<p data-nodeid="2126">因此，我们平时自定义View时，尤其是有可能被ScrollView或者ViewPager嵌套使用的控件，不要遗漏对CANCEL事件的处理，否则有可能引起UI显示异常。</p>
</blockquote>
<h3 data-nodeid="2127">总结</h3>
<p data-nodeid="2128">本课时重点分析了 dispatchTouchEvent 的事件的流程机制，这一过程主要分 3 部分：</p>
<ul data-nodeid="2129">
<li data-nodeid="2130">
<p data-nodeid="2131">判断是否需要拦截 —&gt; 主要是根据 onInterceptTouchEvent 方法的返回值来决定是否拦截；</p>
</li>
<li data-nodeid="2132">
<p data-nodeid="2133">在 DOWN 事件中将 touch 事件分发给子 View —&gt; 这一过程如果有子 View 捕获消费了 touch 事件，会对 mFirstTouchTarget 进行赋值；</p>
</li>
<li data-nodeid="2134">
<p data-nodeid="2135">最后一步，DOWN、MOVE、UP 事件都会根据 mFirstTouchTarget 是否为 null，决定是自己处理 touch 事件，还是再次分发给子 View。</p>
</li>
</ul>
<p data-nodeid="2136">然后介绍了整个事件分发中的几个特殊的点。</p>
<ul data-nodeid="2137">
<li data-nodeid="2138">
<p data-nodeid="2139">DOWN 事件的特殊之处：事件的起点；决定后续事件由谁来消费处理；</p>
</li>
<li data-nodeid="2140">
<p data-nodeid="2141">mFirstTouchTarget 的作用：记录捕获消费 touch 事件的 View，是一个链表结构；</p>
</li>
<li data-nodeid="2142">
<p data-nodeid="2143" class="">CANCEL 事件的触发场景：当父视图先不拦截，然后在 MOVE 事件中重新拦截，此时子 View 会接收到一个 CANCEL 事件。</p>
</li>
</ul>

---

### 精选评论

##### **旋：
> 大型递归函数——典型的责任链模式

##### *准：
> JVM知识比较难啃，Android部分的就简单很多了

##### **福：
> 今天突然发现我之前提的意见 "复制文字会有拉钩的水印的问题" 被处理掉了, 感谢!

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 感谢认可，我们会继续努力的！

##### **代码就跑步：
> 读懂了mFirstTouchTarget，也就看懂了事件分发。

##### **健：
> 很棒

##### **韬：
> 讲得好

##### **山：
> 真棒 谢谢星哥

##### **平：
> 突然发现我一直理解的有一点问题，看来还得好好看看这一块了😹

##### **福：
> 赞赞赞赞赞

