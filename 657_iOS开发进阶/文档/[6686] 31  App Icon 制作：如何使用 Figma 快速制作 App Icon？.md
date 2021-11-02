<p data-nodeid="11145">与后台开发者的要求不一样，iOS 开发者不仅要保证程序逻辑的正确性，而且要注重用户的使用体验。为了给用户带来良好的使用体验，我们需要提升自身的设计能力。那该怎么来提高呢？阅读优秀的开源代码是提升编码能力的有效办法。相应地，学习和模仿优秀设计也是提高设计能力的重要途径。在这一讲中，我会以制作 Moments App 图标（Icon）为例子，讲解如何使用 Figma 进行设计。</p>
<p data-nodeid="11146">目前流行的设计工具主要有 PhotoShop、Adobe XD、Sketch 和 Figma 等，这些工具各有优缺点。而我选择最近一两年特别流行的 Figma，其主要原因有如下。</p>
<ol data-nodeid="11147">
<li data-nodeid="11148">
<p data-nodeid="11149">Figma 能运行在主流的浏览器上，可以在 Mac、Windows、Linux 甚至 Chromebook 下使用，而 Sketch 等一些工具只能在 Mac 上运行。</p>
</li>
<li data-nodeid="11150">
<p data-nodeid="11151">Figma 把所有的设计稿都自动保存在云端，我们可以在任何能连接网络的机器上访问设计稿，无须任何额外的工具和平台来管理设计文件。</p>
</li>
<li data-nodeid="11152">
<p data-nodeid="11153">Figma 提供给个人免费使用，而且不限制文件的数量、云存储的空间，以及预览者和评论者人数等。而其他工具一般都需要收取一定费用，比如，PhotoShop 就收费不菲。</p>
</li>
<li data-nodeid="11154">
<p data-nodeid="11155">Figma 支持多人同时预览、修改和评论同一份设计稿，方便大家在线实时协作，我们可以在设计稿上通过人像实时看到其他同事的操作和留言。</p>
</li>
<li data-nodeid="11156">
<p data-nodeid="11157">Figma 自带版本管理功能，方便我们查看所有人的历史修改记录。</p>
</li>
<li data-nodeid="11158">
<p data-nodeid="11159">Figma 支持按权限级别的方式进行分享，例如设计师可以有修改权限，而产品经理和开发者只具有评论权限。</p>
</li>
<li data-nodeid="11160">
<p data-nodeid="11161">Figma 能生成 Web、iOS 和 Android 的 UI 代码，方便我们把设计稿转换为代码。</p>
</li>
<li data-nodeid="11162">
<p data-nodeid="11163">Figma 支持以 Components（组件）的方式来封装常用的 UI 组件，方便我们把常用组件共享到多个设计稿里面。</p>
</li>
<li data-nodeid="11164">
<p data-nodeid="11165">Figma 为第三方开发者提供了丰富的 API，方便社区开发插件，因此 Figma 具有很多实用的第三方插件来简化我们的设计工作。</p>
</li>
<li data-nodeid="11166">
<p data-nodeid="11167">Figma 支持原型设计功能（Prototyping），这样能帮助我们设计页面之间的交互、换场和导航。</p>
</li>
<li data-nodeid="11168">
<p data-nodeid="11169">Figma 还提供在线预览和演示功能，方便设计师与产品经理、开发者进行实时沟通。</p>
</li>
</ol>
<p data-nodeid="11170">总之，Figma 的功能齐全而且丰富，只需要一个工具就能完成几乎所有的设计工作，不像以前那样，我们需要使用 Sketch 来编辑设计稿，还需要使用 InVision 来共享设计稿，并使用 Marvel 来预览原型等。因此，我非常推荐你使用 Figma，对于个人开发者而言，免费版的功能已经足够满足各个需求了。</p>
<h3 data-nodeid="11171">Figma 工具概括</h3>
<p data-nodeid="11172">那下面我们就一起看看如何使用 Figma 来设计 Moments App 的图标吧。</p>
<p data-nodeid="11173">首先，我们需要在浏览器上登录到 figma.com 网站并免费注册一个账号，然后就会看到 Figma 的主界面，如下图所示：</p>
<p data-nodeid="11982" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/22/CioPOWC9z3OAIUMEAAFI4jh8tqo125.png" alt="Drawing 0.png" data-nodeid="11985"></p>

<p data-nodeid="11175" class="">左边区域是功能菜单，而右边区域是当前选中菜单的详情页面。当我们要开发一个设计稿的时候，第一件事往往是寻找有没有类似模板可以参考。Figma Community 为我们提供了大量的设计模板，可以点击左边的 Community 菜单来打开以下的 Figma 社区页面。</p>
<p data-nodeid="12548" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/1A/Cgp9HWC9z3qATuvCAAI4o7lalO8847.png" alt="Drawing 1.png" data-nodeid="12551"></p>

<p data-nodeid="11177">由于我们要做 App 的图标，因此，可以在搜索框输入“App Icon Template”关键字来搜索 App 图标的模板，结果如下：</p>
<p data-nodeid="13118" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/1A/Cgp9HWC9z4CAEvF4AAOPQ548-S4649.png" alt="Drawing 2.png" data-nodeid="13121"></p>

<p data-nodeid="11179">在实际工作中，我们可以打开多个文件模块进行尝试与比较，并选择自己喜欢的模板。在今天这个例子中，我们选择了图上的“iOS App Icon Template”模板。只需要点击“向下箭头”按钮就能把该模板复制到我们的初稿文件夹里面，复制完毕后会显示下面的提示，翻译成中文就是“文件已经复制到你的初稿里面了”。</p>
<p data-nodeid="14272"><img src="https://s0.lgstatic.com/i/image6/M01/44/23/CioPOWC9z4aAGon8AAAntk0s0Fs586.png" alt="Drawing 3.png" data-nodeid="14276"></p>
<p data-nodeid="14273">点击提示框的“View”按钮就能打开复制的设计稿，如下图所示：</p>



<p data-nodeid="14852" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/1A/Cgp9HWC9z4yADsi1AAFmnSWqxTg642.png" alt="Drawing 4.png" data-nodeid="14855"></p>

<p data-nodeid="11183">这就是我们的工具页面，最上面的黑色部分是菜单栏，我们可以点击左上角的 Figma 图标来打开 Figma 程序的菜单，而 Figma 图标右边就是我们常用的操作，例如添加形状、文本和绘画工具等。</p>
<p data-nodeid="11184">左边区域是设计稿的图层关系组织图，它把当前的页面、图层分组和图层通过树状的方式呈现出来，方便我们快速地定位所需要浏览和编辑的设计元素。</p>
<p data-nodeid="11185">中间区域是主编辑区域，我们的设计工作就在里面完成，例如添加和修改形状、文本等设计元素，改变图层的顺序，移动元素的位置，等等。</p>
<p data-nodeid="11186">右边区域可称为属性配置页，它会根据我们当前选中的设计元素而动态改变，方便我们编辑选中元素的属性。例如，在下图中，当我们选中一个圆形的时候，属性配置页会显示该圆形的位置、约束和填充色等信息，我们可以通过修改这些属性来改变该圆形的设计。</p>
<p data-nodeid="15434" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/1A/Cgp9HWC9z5SAKyW8AAD7-e3IPeo710.png" alt="Drawing 5.png" data-nodeid="15437"></p>

<p data-nodeid="11188">对 Figma 工具有一个概括性的了解以后，下一步我们再来看看如何使用“iOS App Icon Template”模板。</p>
<p data-nodeid="11189">我们需要先详细阅读模板的描述信息，这个模板的描述信息非常简单，如下图所示：</p>
<p data-nodeid="16020" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/23/CioPOWC9z5qAeC_TAAFcNS9pBoQ237.png" alt="Drawing 6.png" data-nodeid="16023"></p>

<p data-nodeid="11191">简单来说，我们需要在名为“icon-ios”的图层里面根据自己的需要设计和编辑 App 的图标，然后把图标的约束（Constraints）设置为“Scale”，最后再把设计好的图标按照图上的各种尺寸进行导出。</p>
<h3 data-nodeid="11192">图标设计</h3>
<p data-nodeid="11193">现在我们就可以开始设计和编辑图标了。为了加快设计的速度，我们可以参考微信朋友圈的图标，该图标可以到微信开放平台的官方文档里找到，如下图所示：</p>
<p data-nodeid="16610" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/1A/Cgp9HWC9z6KAWYC0AACPDYAinvE306.png" alt="Drawing 7.png" data-nodeid="16613"></p>

<p data-nodeid="11195">接着我们就可以下载微信朋友圈图标，并把它放到设计稿里面，如下图所示：</p>
<p data-nodeid="17792" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/23/CioPOWC9z62Abi2xAACcE_JkT6c247.png" alt="Drawing 8.png" data-nodeid="17795"></p>

<h4 data-nodeid="17204" class="">1. 绘制图标</h4>

<p data-nodeid="11200">要学习如何绘制一个图标，第一步可以把该图标里面的元素进行拆分。我们可以看到，该图标的外层是一个圆形，中心部分是一个八边形，中间是八块形状一样但颜色不同的圆边三角形。</p>
<p data-nodeid="11201">拆分完毕以后，我们就可以<strong data-nodeid="11330">从外部的圆形和中心的八边形开始进行编辑</strong>。在“icon-ios”图层里面，我们先把不需要的内圆删除掉，然后点击下图的“Polygon”菜单来添加一个多边形，为了生成等边的多边形，我们需要在拖放该多边形的时候按着 Shift 键。</p>
<p data-nodeid="18386" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/1A/Cgp9HWC9z7WAV0ekAABdx1yh4yU812.png" alt="Drawing 9.png" data-nodeid="18389"></p>

<p data-nodeid="11203">然后在右边的属性配置页中选择长和高都为 20，角度为 22.5 度，边数为 8，这样就能生成一个如下图所示的八边形来。不过需要注意，我们要把这个八边形布局到圆形的中心上，操作也十分简单，只需要点击和拖动即可。</p>
<p data-nodeid="18984" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC9z72AGsF8AACkcS2zrlU932.png" alt="Drawing 10.png" data-nodeid="18987"></p>

<p data-nodeid="11205">接着就可以画一个长方形了，我们可以在菜单里面点击 Rectangle，或者使用快捷键 R 来画，如下图所示：</p>
<p data-nodeid="19586" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/1A/Cgp9HWC9z8SAT31gAADs0RpjNUw198.png" alt="Drawing 11.png" data-nodeid="19589"></p>

<p data-nodeid="11207"><strong data-nodeid="11340">下一步是复制八条形状一样的长方形，并旋转它们的布局角度</strong>。由于 Figma 没有自带的 Sketch 里面的 Rotate Copies（旋转复制）功能，所以我们需要下载一个插件。那怎么寻找和下载第三方插件呢？我们可以点击 Figma -&gt; Plugins -&gt; Browse plugins in Community 菜单来打开社区插件库网页。</p>
<p data-nodeid="20192" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC9z8qAT8pwAADUpZB5nPE746.png" alt="Drawing 12.png" data-nodeid="20195"></p>

<p data-nodeid="11209">然后输入“Rotate Copies”关键字进行搜索，并点击“Install”按钮进行安装，如下图所示：</p>
<p data-nodeid="20802" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC9z9GAd084AAA5E-FP-6A356.png" alt="Drawing 13.png" data-nodeid="20805"></p>

<p data-nodeid="11211">插件安装完毕以后，我们右键图层就能看到如下图所示的插件菜单了。</p>
<p data-nodeid="21416" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC9z9eAdGxnAAEdJTkJemM215.png" alt="Drawing 14.png" data-nodeid="21419"></p>

<p data-nodeid="11213">当我们选择“Rotate Copies”菜单时，Figma 就会弹出下图的弹出框。</p>
<p data-nodeid="22034" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/1A/Cgp9HWC9z-OAfzSJAAD4UbJC3xY413.png" alt="Drawing 15.png" data-nodeid="22037"></p>

<p data-nodeid="11215">该弹出框显示了该插件所需的配置信息，我们可以修改这些配置信息来生成不同的效果。在我们的例子中，我们在“Number of copies”输入框里填入 4，然后点击“Rotate”按钮，这样就能复制 4 条不同角度的长条。由于我们需要 8 条长条，所以还需要再点击一次“Rotate”按钮。</p>
<p data-nodeid="11216">然后把这 8 条长条分别移动到中心八边形的边上，如下图所示：</p>
<p data-nodeid="22656" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/1A/Cgp9HWC9z-qAMWVHAAE8_JE6A9g840.png" alt="Drawing 16.png" data-nodeid="22659"></p>

<p data-nodeid="11218">现在可以删掉中心八边形以及用于辅助布局的十字形状，生成的效果如下图所示：</p>
<p data-nodeid="23282" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC9z_KAHWU9AADkbhxdR0E065.png" alt="Drawing 17.png" data-nodeid="23285"></p>

<p data-nodeid="11220">再下一步就是<strong data-nodeid="11358">把这 8 条长条合并到同一个图层里面</strong>，我们可以在左边区域的图层树上同时选中这 8 条长条，然后右键打开菜单，如下图所示：</p>
<p data-nodeid="23912" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/1A/Cgp9HWC9z_yAGke2AACm8mNcRnw803.png" alt="Drawing 18.png" data-nodeid="23915"></p>

<p data-nodeid="11222">选中 Flatten 菜单项就能合成一个名叫“Vector”的图层了。假如你能熟练使用 Figma，也可以直接通过“Command + E 键”来完成这个合并操作。</p>
<h4 data-nodeid="24546" class="">2. 布尔操作</h4>

<p data-nodeid="26442" class="">有了圆形和 8 条长条以后，怎样才能生成八个圆边三角形呢？这里需要使用到 Boolean Operations（布尔操作）。那什么是布尔操作呢？<strong data-nodeid="26448">布尔操作是把任意数量的形状图层通过 Union（并集）、Subtract（减集）、Intersect（交集）或者 Exclude（差集）方式进行合并。</strong> 为了方便你理解，我就通过下面的图来演示它们的效果：</p>

<p data-nodeid="25808" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/23/CioPOWC90A-AXHTRAAE4RzcdE3Y486.png" alt="Drawing 19.png" data-nodeid="25811"></p>


<p data-nodeid="11228">图最上面展示的是操作前 A 和 B 图层，下面是分别使用四种不同的方式进行布尔操作合并的结果。</p>
<ul data-nodeid="11229">
<li data-nodeid="11230">
<p data-nodeid="11231">Union 会把两个图层合并在同一个图层，重复部分的轮廓线会被抹掉，只保留外部的轮廓线。</p>
</li>
<li data-nodeid="11232">
<p data-nodeid="11233">Subtract 和 Union 相反，该操作需要制定基础图层。由于 A 是基础图层，Subtract 操作会从 A 图层去掉 B 图层部分，并只保留基础图层中没有被覆盖的部分。</p>
</li>
<li data-nodeid="11234">
<p data-nodeid="11235">Intersect 只保留两个图层的重复部分。</p>
</li>
<li data-nodeid="11236">
<p data-nodeid="11237">Exclude 和 Intersect 相反，会把重复部分去掉，只保留不重复的部分。</p>
</li>
</ul>
<p data-nodeid="11238">合理使用这四个布尔操作能帮助我们绘制各种效果的图标，比如，现如今流行的 App 图标，其中绝大部分图标都可以通过图层间的布尔操作来制作。</p>
<p data-nodeid="11239">下面我们就一起看看如何使用布尔操作来制作八个圆边三角形。首先需要同时选中“Vector”（八条边的合并图）和“Ellipse”（圆形）两个图层，然后点击下图的“Subtract Selection”菜单。</p>
<p data-nodeid="27081" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC90IyAKRPHAAG0Y4_bhXU110.png" alt="Drawing 20.png" data-nodeid="27084"></p>

<p data-nodeid="11241">这样就能生成如下图所示的八个圆边三角形了。</p>
<p data-nodeid="27721" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/1B/Cgp9HWC90JWAVyhIAAFzRizSj7U520.png" alt="Drawing 21.png" data-nodeid="27724"></p>

<h4 data-nodeid="28365" class="">3. 分离图层</h4>

<p data-nodeid="11246">因为布尔操作会把多个图层合并成单独一个图层，所以我们没有办法为各个圆边三角形分别添加不同的颜色。那怎么办呢？我告诉你一个小技巧：首先把图层先通过 Flatten 进行合并，然后导出成 SVG 文件，再把 SVG 文件拖进设计稿进行编辑。</p>
<p data-nodeid="11247">前面已经讲过，我们可以通过“Command + E 键”来合并成如下图的“Subtract”的图层。导出 SVG 的操作也非常简单，先选择需要导出的图层，再打开右边的 Export 配置，最后把导出的文件格式修改为 SVG，并点击“Export Subtract”按钮即可。</p>
<p data-nodeid="29003" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/1B/Cgp9HWC90KGAUsZbAAEudY4FVjo855.png" alt="Drawing 22.png" data-nodeid="29006"></p>

<p data-nodeid="11249">现在就可以在“icon-ios”里把原有的“Subtract”图层删除掉，并把导出的 SVG 文件拖进里面。如下图所示，你看到一个名叫“Exported Subtract”的新图层。</p>
<p data-nodeid="29647" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC90KiAKgZiAADstdxYsYc335.png" alt="Drawing 23.png" data-nodeid="29650"></p>

<p data-nodeid="11251">当我们设计完一个图标以后，通常会使用到 Home Screen、App Store 和 Push Notification 等不同的地方，各个地方图标的尺寸各不相同，为了能正确地导出各种尺寸的图标，我们必须在属性配置页上把 Constraints 设置为 Scale。这一步非常重要，初次使用 Figma 常常容易忽略这一细节。</p>
<p data-nodeid="11252">接着可以删掉不需要的图层，如下图所示，我们可以展开“Exported Subtract”图层并选中第一、第二个图层。</p>
<p data-nodeid="31603" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC90OaAbSeFAABQV0Sa8cU177.png" alt="Drawing 24.png" data-nodeid="31606"></p>


<p data-nodeid="11254">把它们删掉后，就只保留了八个圆边三角形，如下图所示：</p>
<p data-nodeid="30947" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC90NyAQFlhAADVT10WJ-k599.png" alt="Drawing 25.png" data-nodeid="30950"></p>

<h4 data-nodeid="32259" class="">4. 填充颜色</h4>

<p data-nodeid="11259">最后一步是<strong data-nodeid="11395">为各个圆边三角形分别填充颜色</strong>。填充颜色的操作非常简单，先选择其中一个圆边三角形，然后点击 Control + C 把鼠标变成取色器，接着再把取色器移动到右边的图就能撷取各块的颜色了，如下图所示：</p>
<p data-nodeid="32909" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC90PCAKg64AAENBx1xXDY270.png" alt="Drawing 26.png" data-nodeid="32912"></p>

<p data-nodeid="11261">就这样，八个圆边三角形填充完毕以后效果如下：</p>
<p data-nodeid="33565" class=""><img src="https://s0.lgstatic.com/i/image6/M01/44/1B/Cgp9HWC90PaANXn_AAD-vsRdji4260.png" alt="Drawing 27.png" data-nodeid="33568"></p>

<p data-nodeid="11263">同时，我们还可以看到该图标应用到不同地方的效果，如下图所示：</p>
<p data-nodeid="34225" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC90P2AYjVzAAJ7qHdIOBA753.png" alt="Drawing 28.png" data-nodeid="34228"></p>

<h4 data-nodeid="34889" class="">5. 导出图标</h4>

<p data-nodeid="11268">接着就可以导出图标了，选择“icon-ios”图层并点击“Export icon-ios”按钮，如下图所示：</p>
<p data-nodeid="35547" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/23/CioPOWC90QWAISIHAADw4hSpf38419.png" alt="Drawing 29.png" data-nodeid="35550"></p>

<p data-nodeid="11270">最后一步就是把图标导入 Xcode 里面，只需把所有的 PNG 文件拖到 Xcode 的 AppIcons 下就完成了。</p>
<p data-nodeid="36211" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/44/1B/Cgp9HWC90QuAUXIaAAFLc1rV-Fs236.png" alt="Drawing 30.png" data-nodeid="36214"></p>

<h3 data-nodeid="11272">总结</h3>
<p data-nodeid="11273">在这一讲，我们介绍了如何使用 Figma 来绘制一个 Moments App 的图标。其中，我们重点讲解了如何寻找模板、怎样安装插件以及如何使用布尔操作来合并多个图层。</p>
<p data-nodeid="11274">如果你刚入门 App 设计，我建议你从模仿图标的制作入手，把流行的 App 图标下载下来，思考如何把图标拆解成基本图形，例如圆形和多边形等，然后通过布尔操作把多个图形组合成新图层，并填充颜色。一旦能熟练设计图标，页面的设计就变得相对简单了。学习页面设计的方法和图标设计类似，可以通过学习流行的 App 以及 Dribbble 上的热门设计来不断提高。当你掌握这些后，还可以系统地学习字体、颜色、间距、对比度、布局、信息架构等内容。</p>
<p data-nodeid="11275">在设计工具的选择上，Figma 简单易用，功能齐全，而且支持团队协作，我非常建议你多花一点时间学习与研究，这会很好地帮助你与设计师进行沟通和交流。</p>
<p data-nodeid="11276"><strong data-nodeid="11413">思考题</strong></p>
<blockquote data-nodeid="11277">
<p data-nodeid="11278">今天留给你一个操作题：请使用 Figma 绘制一个微信 App 的图标。</p>
</blockquote>
<p data-nodeid="11279">可以把你的设计稿发到留言区哦。下一讲我将介绍如何使用 SwiftUI 快速替换原有 UI 层，记得按时来听课。</p>
<p data-nodeid="11280"><strong data-nodeid="11419">源码地址</strong></p>
<blockquote data-nodeid="11281">
<p data-nodeid="11282">App 图标设计稿地址：<a href="https://www.figma.com/file/Zn86wXELmljrYKZKu95uKf/iOS-App-Icon-Template-Community?node-id=1:55&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="11423">https://www.figma.com/file/Zn86wXELmljrYKZKu95uKf/iOS-App-Icon-Template-Community?node-id=1%3A55</a></p>
</blockquote>

---

### 精选评论


