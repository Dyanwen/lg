<p>本课时我们主要学习 Android 是如何自定义 View 的。</p>
<p>当 Android SDK 中提供的系统 UI 控件无法满足业务需求时，我们就需要考虑自己实现 UI 控件。另外掌握自定义控件，也是理解整套 Android 渲染体系的基础，它能够很好地体现一个 Android 工程师对操作系统的理解深度，因此这部分知识也是面试时经常被问到的知识点。关于 UI 渲染的详细流程将在第 21 课时详细介绍。</p>
<p>自定义 UI 控件有 2 种方式：</p>
<ol>
<li>继承系统提供的成熟控件（比如 LinearLayout、RelativeLayout、ImageView 等）；</li>
<li>直接继承自系统 View 或者 ViewGroup，并自绘显示内容。</li>
</ol>
<h3>继承现有控件</h3>
<p>相对而言，这是一种较简单的实现方式。因为大部分核心工作，比如关于控件大小的测量、控件位置的摆放等相关的计算，在系统中都已经实现并封装好，开发人员只要在其基础上进行一些扩展，并按照自己的意图显示相应的 UI 元素。比如以下代码：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/Ciqc1F66bomAdFbwAALM9ajQIC8076.png" alt="image.png"></p>
<p>CustomToolBar 继承自 RelativeLayout，在构造函数中通过 addView 方式分别添加了 2 个 ImageView 和 1 个 TextView。显示效果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/CgqCHl66bpKAIlBcAAA0F5H64pY243.png" alt="image (1).png"></p>
<h3>自定义属性</h3>
<p>有时候我们想在 XML 布局文件中使用 CustomToolBar 时，希望能在 XML 文件中直接指定 title 的显示内容、字体颜色，leftImage 和 rightImage 的显示图片等。这就需要使用自定义属性。<br>
自定义属性具体步骤分为以下几步：</p>
<h4>attrs.xml 中声明自定义属性</h4>
<p>在 res 的 values 目录下的 attrs.xml 文件中（没有就自己新建一个），使用 <declare-styleable> 标签自定义属性，如下所示：</declare-styleable></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/Ciqc1F66bpuAIr0aAAFqNwcLsJ0889.png" alt="image (2).png"></p>
<p>解释说明：</p>
<ul>
<li><declare-styleable> 标签代表定义一个自定义属性集合，一般会与自定义控件结合使用；</declare-styleable></li>
<li><attr> 标签则是某一条具体的属性，name 是属性名称，format 代表属性的格式。</attr></li>
</ul>
<h4>在 XML 布局文件中使用自定义属性</h4>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/CgqCHl66bqWAFxZfAAKl9XNB380067.png" alt="image (3).png"></p>
<p>需要先添加命名空间 xmlns:app，然后通过命名空间 app 引用自定义属性，并传入相应的图片资源和字符串内容。</p>
<h4>在 CustomToolBar 中，获取自定义属性的引用值</h4>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/CgqCHl66bq6AU-oEAAIh3vx_5eM610.png" alt="image (4).png"></p>
<p>如上图所示，主要是通过 Context.obtainStyleAttributes 方法获取到自定义属性的集合，然后从这个集合中取出相应的自定义属性。</p>
<p>完整代码参考：<a href="https://github.com/McoyJiang/LagouAndroidShare/tree/master/course15_%E8%87%AA%E5%AE%9A%E4%B9%89View/LagouCustomizedView">拉勾课程代码仓库 课时15</a></p>
<h3>直接继承自 View 或者 ViewGroup</h3>
<p>这种方式相比第一种麻烦一些，但是更加灵活，也能实现更加复杂的 UI 界面。一般情况下使用这种实现方式需要解决以下几个问题：</p>
<ol>
<li>如何根据相应的属性将 UI 元素绘制到界面；</li>
<li>自定义控件的大小，也就是宽和高分别设置多少；</li>
<li>如果是 ViewGroup，如何合理安排其内部子 View 的摆放位置。</li>
</ol>
<p>以上 3 个问题依次在如下 3 个方法中得到解决：</p>
<ol>
<li>onDraw</li>
<li>onMeasure</li>
<li>onLayout</li>
</ol>
<p>因此自定义 View 的重点工作其实就是复写并合理的实现这 3 个方法。注意：并不是每个自定义 View 都需要实现这 3 个方法，大多数情况下只需要实现其中 2 个甚至 1 个方法也能满足需求。</p>
<h4>onDraw</h4>
<p>onDraw 方法接收一个 Canvas 类型的参数。Canvas 可以理解为一个画布，在这块画布上可以绘制各种类型的 UI 元素。</p>
<p>系统提供了一系列 Canvas 操作方法，如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/Ciqc1F66brqANYwaAAFgenmfG7o790.png" alt="image (5).png"></p>
<p>从上图中可以看出，Canvas 中每一个绘制操作都需要传入一个 Paint 对象。Paint 就相当于一个画笔，我们可以通过设置画笔的各种属性，来实现不同绘制效果：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/CgqCHl66bsKAC3aYAAEfignRLSI590.png" alt="image (6).png"></p>
<p>比如如下代码，定义 PieImageView 继承自 View，然后在 onDraw 方法中，分别使用 Canvas 的 drawArc 和 drawCircle 方法来绘制弧度和圆形。这两个形状组合在一起就能表示一个简易的圆形进度条控件。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/Ciqc1F66bs2AS0zEAAQY2ssC74o325.png" alt="image (7).png"></p>
<p>在布局文件中直接使用上述的 PieImageView，设置宽高为 300dp，并在 Activity 中设置 PieImageView 的进度为 45，如下所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/CgqCHl66btSAYWPrAAH9B9MVFqA640.png" alt="image (8).png"></p>
<p>最终运行显示效果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/Ciqc1F66btyAc0PJAAIpVraUjo0218.png" alt="image (9).png"></p>
<p>如果在上面代码中的布局文件中，将 PieImageView 的宽高设置为 wrap_content（也就是自适应），重新运行则显示效果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/CgqCHl66buWAP4coAAG7qEuu7jo510.png" alt="image (10).png"></p>
<p>很显然，PieImageView 并没有正常显示。问题的主要原因就是在 PieImageView 中并没有在 onMeasure 方法中进行重新测量，并重新设置宽高。</p>
<h4>onMeasure</h4>
<p>首先我们需要弄清楚，自定义 View 为什么需要重新测量。正常情况下，我们直接在 XML 布局文件中定义好 View 的宽高，然后让自定义 View 在此宽高的区域内显示即可。但是为了更好地兼容不同尺寸的屏幕，Android 系统提供了 wrap_contetn 和 match_parent 属性来规范控件的显示规则。它们分别代表<strong>自适应大小</strong>和<strong>填充父视图</strong>的<strong>大小</strong>，但是这两个属性并没有指定具体的大小，因此我们需要在 onMeasure 方法中过滤出这两种情况，真正的测量出自定义 View 应该显示的宽高大小。</p>
<p>所有工作都是在 onMeasure 方法中完成，方法定义如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/CgqCHl66bvCAUiTLAACDybKUm44275.png" alt="image (11).png"></p>
<p>可以看出，方法会传入 2 个参数 widthMeasureSpec 和 heightMeasureSpec。这两个参数是从父视图传递给子 View 的两个参数，看起来很像宽、高，但是它们所表示的不仅仅是宽和高，还有一个非常重要的<strong>测量模式</strong>。</p>
<p>一共有 3 种测量模式。</p>
<ol>
<li>EXACTLY：表示在 XML 布局文件中宽高使用 match_parent 或者固定大小的宽高；</li>
<li>AT_MOST：表示在 XML 布局文件中宽高使用 wrap_content；</li>
<li>UNSPECIFIED：父容器没有对当前 View 有任何限制，当前 View 可以取任意尺寸，比如 ListView 中的 item。</li>
</ol>
<p>具体值和测量模式都可以通过 Android SDK 中提供的 MeasureSpec.java 类获取：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/CgqCHl66bvqADG9BAADvBFUSQJk100.png" alt="image (12).png"></p>
<p>为什么 1 个 int 值可以代表 2 种意义呢？ 实际上 widthMeasureSpec 和 heightMeasureSpec 都是使用二进制高 2 位表示测量模式，低 30 位表示宽高具体大小。</p>
<p><strong>重新回到 PieImageView</strong></p>
<p>在 PieImageView 中并没有复写 onMeasure 方法，因此默认使用父类也就是 View 中的实现，View 中的 onMeasure 默认实现如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/47/CgqCHl66bw-ADT0MAADKTPdZQ1c840.png" alt="image (13).png"></p>
<p>蓝色框中的 <strong>setMeasuredDimension</strong> 是一个非常重要的方法，这个方法传入的值直接决定 View 的宽高，也就是说如果调用 setMeasuredDimension(100,200)，最终 View 就显示宽 100 *  高 200 的矩形范围。红色下划线标识的 getDefaultSize 返回的是默认大小，默认为父视图的剩余可用空间。</p>
<blockquote>
<p>这也是为什么 PieImageView 显示异常的原因，虽然我们在 XML 中指定的是 wrap_content，但是实际使用的宽高值却是父视图的剩余可用空间，从代码中可以看出是整个屏幕的宽高。</p>
</blockquote>
<p>问题的原因找到，解决方法只要复写 onMeasure，过滤出 wrap_content 的情况，并主动调用 setMeasuredDimension 方法设置正确的宽高即可：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/48/CgqCHl66bxmAZbnFAAH8BgmvD9Q604.png" alt="image (14).png"></p>
<p><strong>ViewGroup 中的 onMeasure</strong></p>
<p>如果我们自定义的控件是一个容器，onMeasure 方法会更加复杂一些。因为 ViewGroup 在测量自己的宽高之前，需要先确定其内部子 View 的所占大小，然后才能确定自己的大小。比如如下一段代码：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/48/Ciqc1F66byKARzSmAAc9fHTIEMg900.png" alt="image (15).png"></p>
<p>LinearLayout 的宽高为 wrap_content 表示由子控件的大小决定，而 3 个子控件的宽度分别为300、200、100，那最终 LinearLayout 的宽度显示多少呢？ 运行结果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/48/CgqCHl66byqAFve6AAAqPxvb9sA095.png" alt="image (16).png"></p>
<p>可以看出 LinearLayout 的最终宽度由其内部最大的子 View 宽度决定。</p>
<p>当我们自己定义一个 ViewGroup 的时候，也需要在 onMeasure 方法中综合考虑子 View 的宽度。比如如果要实现一个流式布局 FlowLayout，效果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/48/Ciqc1F66b0uANdyTAASLs9Xvo14469.png" alt="image (17).png"></p>
<p>在大多数 App 的搜索界面经常会使用 FlowLayout 来展示历史搜索记录或者热门搜索项。<br>
FlowLayout 的每一行上的 item 个数不一定，当每行的 item 累计宽度超过可用总宽度，则需要重启一行摆放 item 项。因此我们需要在 onMeasure 方法中主动的分行计算出 FlowLayout 的最终高度，如下所示：</p>
<pre><code data-language="java" class="lang-java">   <span class="hljs-comment">//测量控件的宽和高</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onMeasure</span><span class="hljs-params">(<span class="hljs-keyword">int</span> widthMeasureSpec, <span class="hljs-keyword">int</span> heightMeasureSpec)</span> </span>{
        <span class="hljs-keyword">super</span>.onMeasure(widthMeasureSpec, heightMeasureSpec);
        <span class="hljs-comment">//获得宽高的测量模式和测量值</span>
        <span class="hljs-keyword">int</span> widthMode = MeasureSpec.getMode(widthMeasureSpec);
        <span class="hljs-keyword">int</span> widthSize = MeasureSpec.getSize(widthMeasureSpec);
        <span class="hljs-keyword">int</span> heightSize = MeasureSpec.getSize(heightMeasureSpec);
        <span class="hljs-keyword">int</span> heightMode = MeasureSpec.getMode(heightMeasureSpec);

        <span class="hljs-comment">//获得容器中子View的个数</span>
        <span class="hljs-keyword">int</span> childCount = getChildCount();
        <span class="hljs-comment">//记录每一行View的总宽度</span>
        <span class="hljs-keyword">int</span> totalLineWidth = <span class="hljs-number">0</span>;
        <span class="hljs-comment">//记录每一行最高View的高度</span>
        <span class="hljs-keyword">int</span> perLineMaxHeight = <span class="hljs-number">0</span>;
        <span class="hljs-comment">//记录当前ViewGroup的总高度</span>
        <span class="hljs-keyword">int</span> totalHeight = <span class="hljs-number">0</span>;
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; childCount; i++) {
            View childView = getChildAt(i);
            <span class="hljs-comment">//对子View进行测量</span>
            measureChild(childView, widthMeasureSpec, heightMeasureSpec);
            MarginLayoutParams lp = (MarginLayoutParams) childView.getLayoutParams();
            <span class="hljs-comment">//获得子View的测量宽度</span>
            <span class="hljs-keyword">int</span> childWidth = childView.getMeasuredWidth() + lp.leftMargin + lp.rightMargin;
            <span class="hljs-comment">//获得子View的测量高度</span>
            <span class="hljs-keyword">int</span> childHeight = childView.getMeasuredHeight() + lp.topMargin + lp.bottomMargin;
            <span class="hljs-keyword">if</span> (totalLineWidth + childWidth &gt; widthSize) {
                <span class="hljs-comment">//统计总高度</span>
                totalHeight += perLineMaxHeight;
                <span class="hljs-comment">//开启新的一行</span>
                totalLineWidth = childWidth;
                perLineMaxHeight = childHeight;
            } <span class="hljs-keyword">else</span> {
                <span class="hljs-comment">//记录每一行的总宽度</span>
                totalLineWidth += childWidth;
                <span class="hljs-comment">//比较每一行最高的View</span>
                perLineMaxHeight = Math.max(perLineMaxHeight, childHeight);
            }
            <span class="hljs-comment">//当该View已是最后一个View时，将该行最大高度添加到totalHeight中</span>
            <span class="hljs-keyword">if</span> (i == childCount - <span class="hljs-number">1</span>) {
                totalHeight += perLineMaxHeight;
            }
        }
        <span class="hljs-comment">//如果高度的测量模式是EXACTLY，则高度用测量值，否则用计算出来的总高度（这时高度的设置为wrap_content）</span>
        heightSize = heightMode == MeasureSpec.EXACTLY ? heightSize : totalHeight;
        setMeasuredDimension(widthSize, heightSize);
    }
</code></pre>
<p>上述 onMeasure 方法的主要目的有 2 个：</p>
<ul>
<li>调用 measureChild 方法递归测量子 View；</li>
<li>通过叠加每一行的高度，计算出最终 FlowLayout 的最终高度 totalHeight。</li>
</ul>
<h4>onLayout</h4>
<p>上面的 FlowLayout 中的 onMeasure 方法只是计算出 ViewGroup 的最终显示宽高，但是并没有规定某一个子 View 应该显示在何处位置。要定义 ViewGroup 内部子 View 的显示规则，则需要复写并实现 onLayout 方法。</p>
<p>ViewGroup 中的 onLayout 方法声明如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/48/Ciqc1F66b1eAeNsoAABs2Q0QRN0512.png" alt="image (18).png"></p>
<p>它是一个抽象方法，也就是说每一个自定义 ViewGroup 都必须主动实现如何排布子 View，具体就是遍历每一个子 View，调用 child.(l, t, r, b) 方法来为每个子 View 设置具体的布局位置。四个参数分别代表左上右下的坐标位置，一个简易的 FlowLayout 实现如下：</p>
<pre><code data-language="java" class="lang-java">    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onLayout</span><span class="hljs-params">(<span class="hljs-keyword">boolean</span> changed, <span class="hljs-keyword">int</span> l, <span class="hljs-keyword">int</span> t, <span class="hljs-keyword">int</span> r, <span class="hljs-keyword">int</span> b)</span> </span>{
        mAllViews.clear();
        mPerLineMaxHeight.clear();
        <span class="hljs-comment">//存放每一行的子View</span>
        List&lt;View&gt; lineViews = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
        <span class="hljs-comment">//记录每一行已存放View的总宽度</span>
        <span class="hljs-keyword">int</span> totalLineWidth = <span class="hljs-number">0</span>;
        <span class="hljs-comment">//记录每一行最高View的高度</span>
        <span class="hljs-keyword">int</span> lineMaxHeight = <span class="hljs-number">0</span>;
        <span class="hljs-comment">/****遍历所有View，将View添加到List&lt;List&lt;View&gt;&gt;集合中**********/</span>
        <span class="hljs-comment">//获得子View的总个数</span>
        <span class="hljs-keyword">int</span> childCount = getChildCount();
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; childCount; i++) {
            View childView = getChildAt(i);
            MarginLayoutParams lp = (MarginLayoutParams) childView.getLayoutParams();
            <span class="hljs-keyword">int</span> childWidth = childView.getMeasuredWidth() + lp.leftMargin + lp.rightMargin;
            <span class="hljs-keyword">int</span> childHeight = childView.getMeasuredHeight() + lp.topMargin + lp.bottomMargin;
            <span class="hljs-keyword">if</span> (totalLineWidth + childWidth &gt; getWidth()) {
                mAllViews.add(lineViews);
                mPerLineMaxHeight.add(lineMaxHeight);
                <span class="hljs-comment">//开启新的一行</span>
                totalLineWidth = <span class="hljs-number">0</span>;
                lineMaxHeight = <span class="hljs-number">0</span>;
                lineViews = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
            }
            totalLineWidth += childWidth;
            lineViews.add(childView);
            lineMaxHeight = Math.max(lineMaxHeight, childHeight);
        }
        <span class="hljs-comment">//单独处理最后一行</span>
        mAllViews.add(lineViews);
        mPerLineMaxHeight.add(lineMaxHeight);
        <span class="hljs-comment">/************遍历集合中的所有View并显示出来************/</span>
        <span class="hljs-comment">//表示一个View和父容器左边的距离</span>
        <span class="hljs-keyword">int</span> mLeft = <span class="hljs-number">0</span>;
        <span class="hljs-comment">//表示View和父容器顶部的距离</span>
        <span class="hljs-keyword">int</span> mTop = <span class="hljs-number">0</span>;
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; mAllViews.size(); i++) {
            <span class="hljs-comment">//获得每一行的所有View</span>
            lineViews = mAllViews.get(i);
            lineMaxHeight = mPerLineMaxHeight.get(i);
            <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> j = <span class="hljs-number">0</span>; j &lt; lineViews.size(); j++) {
                View childView = lineViews.get(j);
                MarginLayoutParams lp = (MarginLayoutParams) childView.getLayoutParams();
                <span class="hljs-keyword">int</span> leftChild = mLeft + lp.leftMargin;
                <span class="hljs-keyword">int</span> topChild = mTop + lp.topMargin;
                <span class="hljs-keyword">int</span> rightChild = leftChild + childView.getMeasuredWidth()；
                <span class="hljs-keyword">int</span> bottomChild = topChild + childView.getMeasuredHeight();
                <span class="hljs-comment">//四个参数分别表示View的左上角和右下角</span>
                childView.layout(leftChild, topChild, rightChild, bottomChild);
                mLeft += lp.leftMargin + childView.getMeasuredWidth() + lp.rightMargin;
            }
            mLeft = <span class="hljs-number">0</span>;
            mTop += lineMaxHeight;
        }
    }
</code></pre>
<p>最终我们可以在 XML 中使用此自定义控件：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/48/Ciqc1F66b3CAEEa5AAUEZe6Z528059.png" alt="image (19).png"></p>
<p>一个简易的 FlowLayout 运行效果如下所示，剩下的就是和 UI 同事合作，修改 FlowLayout 内部 TextView 的样式，将界面调整到最佳显示状态。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/08/48/CgqCHl66b4SAPQ8iAADyh9KR7Ng588.png" alt="image (20).png"></p>
<h3>总结</h3>
<p>本课时介绍了自定义 View 的几个知识点，要自定义一个控件主要包含几个方法。<br>
onDraw：主要负责绘制 UI 元素；<br>
onMeasure：主要负责测量自定义控件具体显示的宽高；<br>
onLayout：主要是在自定义 ViewGroup 中复写，并实现子 View 的显示位置，并在其中介绍了自定义属性的使用方法。</p>
<p>所有代码都已经提交到：<a href="https://github.com/McoyJiang/LagouAndroidShare/tree/master/course15_%E8%87%AA%E5%AE%9A%E4%B9%89View/LagouCustomizedView">拉勾课程代码仓库 课时15</a></p>

---

### 精选评论

##### **昌：
> 天天追

##### **航：
> 感觉很清晰，突出重点

##### *川：
> 写的通俗易懂，点赞！

##### **用户4504：
> 终于看到明白的自定义view了😂

##### **堂：
> 把onMeasure方法又学习了一遍

