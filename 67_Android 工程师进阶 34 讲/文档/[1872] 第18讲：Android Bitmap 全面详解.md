<p>本课时我们主要对 Bitmap 进行详解。</p>
<p>每一个 Android App 中都会使用到 Bitmap，它也是程序中内存消耗的大户，当 Bitmap 使用内存超过可用空间，则会报 OOM。 因此如何正确使用也是 Android 工程师的重点关注内容。</p>
<h3>Bitmap 占用内存分析</h3>
<p>Bitmap 用来描述一张图片的长、宽、颜色等信息。通常情况下，我们可以使用 BitmapFactory 来将某一路径下的图片解析为 Bitmap 对象。</p>
<p>当一张图片加载到内存后，具体需要占用多大内存呢？</p>
<h4>getAllocationByteCount 探索</h4>
<p>我们可以通过 Bitmap.getAllocationByteCount() 方法获取 Bitmap 占用的字节大小，比如以下代码：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AD/CgqCHl7GI9-AU6LJAAFunSI1GAc025.png" alt="image.png"></p>
<p>上图中 rodman 是保存在 res/drawable-xhdpi 目录下的一张 600*600，大小为 65Kb 的图片。打印结果如下：</p>
<blockquote>
<p>I/Bitmap&nbsp;&nbsp;( 5673): bitmap size is <strong>1440000</strong></p>
</blockquote>
<p><strong>解释</strong></p>
<p>默认情况下 BitmapFactory 使用&nbsp;Bitmap.Config.ARGB_8888 的存储方式来加载图片内容，而在这种存储模式下，每一个像素需要占用 4 个字节。因此上面图片 rodman 的内存大小可以使用如下公式来计算：</p>
<blockquote>
<p>宽 * 高 * 4 = 600 * 600 * 4 = <strong>1440000</strong></p>
</blockquote>
<h4>屏幕自适应</h4>
<p>但是如果我们在保证代码不修改的前提下，将图片 rodman 移动到（注意是移动，不是拷贝）res/drawable-hdpi 目录下，重新运行代码，则打印日志如下：</p>
<blockquote>
<p>I/Bitmap&nbsp;&nbsp;( 6047): bitmap size is 2560000</p>
</blockquote>
<p>可以看出我们只是移动了图片的位置，Bitmap 所占用的空间竟然上涨了 77%。这是为什么呢？</p>
<p>实际上 BitmapFactory 在解析图片的过程中，会根据<strong>当前设备屏幕密度</strong>和<strong>图片所在的 drawable 目录</strong>来做一个对比，根据这个对比值进行缩放操作。具体公式为如下所示：</p>
<ol>
<li>缩放比例 scale = 当前设备屏幕密度 / 图片所在 drawable 目录对应屏幕密度</li>
<li>Bitmap 实际大小 = 宽 * scale * 高 * scale * Config 对应存储像素数</li>
</ol>
<p>在 Android 中，各个 drawable 目录对应的屏幕密度分别为下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/B1/CgqCHl7GJ4CAJWx9AACWL0y3jm0873.png" alt="111.png"></p>
<p>我运行的设备是 Nexus 4，屏幕密度为 320。如果将 rodman 放到 drawable-hdpi 目录下，最终的计算公式如下：</p>
<blockquote>
<p>rodman 实际占用内存大小 = 600 * (320 / 240) * 600 * (320 / 240) * 4 = <strong>2560000</strong></p>
</blockquote>
<h4>assets 中的图片大小</h4>
<p>我们知道，Android 中的图片不仅可以保存在 drawable 目录中，还可以保存在 assets 目录下，然后通过 AssetManager 获取图片的输入流。那这种方式加载生成的 Bitmap 是多大呢？同样是上面的 rodman.png，这次将它放到 assets 目录中，使用如下代码加载：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AD/CgqCHl7GI-6ADafPAAC87wB3U2c974.png" alt="image (1).png"></p>
<p>最终打印结果如下：</p>
<blockquote>
<p>I/Bitmap&nbsp;&nbsp;( 5673): bitmap size is <strong>1440000</strong></p>
</blockquote>
<p>可以看出，加载 assets 目录中的图片，系统并不会对其进行缩放操作。</p>
<h3>Bitmap 加载优化</h3>
<p>上面的例子也能看出，一张 65Kb 大小的图片被加载到内存后，竟然占用了 2560000 个字节，也就是 2.5M 左右。因此适当时候，我们需要对需要加载的图片进行缩略优化。</p>
<p><strong>修改图片加载的 Config</strong></p>
<p>修改占用空间少的存储方式可以快速有效降低图片占用内存。比如通过 BitmapFactory.Options 的 inPreferredConfig 选项，将存储方式设置为&nbsp;Bitmap.Config.RGB_565。这种存储方式一个像素占用 2 个字节，所以最终占用内存直接减半。如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AE/CgqCHl7GJUyAPcYTAAHDKDnB6wE358.png" alt="image (2).png"></p>
<p>打印日志如下:</p>
<blockquote>
<p>I/Bitmap&nbsp;&nbsp;( 6339): bitmap size is <strong>720000</strong></p>
</blockquote>
<p>另外 Options 中还有一个 inSampleSize 参数，可以实现 Bitmap 采样压缩，这个参数的含义是宽高维度上每隔 inSampleSize 个像素进行一次采集。比如以下代码：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A3/Ciqc1F7GJVyActI9AAIFtHMpSCo369.png" alt="image (3).png"></p>
<p>因为宽高都会进行采样，所以最终图片会被缩略 4 倍，最终打印效果如下：</p>
<blockquote>
<p>I/Bitmap&nbsp;&nbsp;( 6414): bitmap size is <strong>180000</strong> &nbsp; // 170Kb</p>
</blockquote>
<h3>Bitmap 复用</h3>
<h4>场景描述</h4>
<p>如果在 Android 某个页面创建很多个 Bitmap，比如有两张图片 A 和 B，通过点击某一按钮需要在 ImageView 上切换显示这两张图片，实现效果如下所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A3/Ciqc1F7GJYyACIiCAMbF_4x1vIQ360.gif" alt="image.gif"></p>
<p>可以使用以下代码实现上述效果：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AF/CgqCHl7GJaqABz77AAIZRUExynU013.png" alt="image (4).png"></p>
<p>但是在每次调用 switchImage 切换图片时，都需要通过 BitmapFactory 创建一个新的 Bitmap 对象。当方法执行完毕后，这个 Bitmap 又会被 GC 回收，这就造成不断地创建和销毁比较大的内存对象，从而导致频繁 GC（或者叫内存抖动）。像 Android App 这种面相最终用户交互的产品，如果因为频繁的 GC 造成 UI 界面卡顿，还是会影响到用户体验的。可以在 Android Studio Profiler 中查看内存情况，多次切换图片后，显示的效果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A3/Ciqc1F7GJbGAX18WAAJGL3irS4A779.png" alt="image (5).png"></p>
<h4>使用 Options.inBitmap 优化</h4>
<p>实际上经过第一次显示之后，内存中已经存在了一个 Bitmap 对象。每次切换图片只是显示的内容不一样，我们可以重复利用已经占用内存的 Bitmap 空间，具体做法就是使用 Options.inBitmap 参数。将 getBitmap 方法修改如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AF/CgqCHl7GJbmAaThsAAfZxD2Nk4g697.png" alt="image (6).png"></p>
<p>解释说明：</p>
<ul>
<li>图中 1 处创建一个可以用来复用的 Bitmap 对象。</li>
<li>图中 2 处，将 options.inBitmap 赋值为之前创建的 reuseBitmap 对象，从而避免重新分配内存。</li>
</ul>
<p>重新运行代码，并查看 Profiler 中的内存情况，可以发现不管我们切换图片多少次，内存占用始终处于一个水平线状态。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A3/Ciqc1F7GJcCARDsaAAB-hGb9K9w827.png" alt="image (7).png"></p>
<p><strong>注意</strong>：在上述 getBitmap 方法中，复用 inBitmap 之前，需要调用 canUseForInBitmap 方法来判断 reuseBitmap 是否可以被复用。这是因为 Bitmap 的复用有一定的限制：</p>
<ul>
<li>在 Android 4.4 版本之前，只能重用相同大小的 Bitmap 内存区域；</li>
<li>4.4 之后你可以重用任何 Bitmap 的内存区域，只要这块内存比将要分配内存的 bitmap 大就可以。</li>
</ul>
<p>canUserForInBitmap 方法具体如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AF/CgqCHl7GJciALgl-AAJID6lRHu4721.png" alt="image (8).png"></p>
<p>细心的你可能也发现了在每次加载之前，除了 inBitmap 参数之外，我还将 Options.inMutable 置为 true，这里如果不置为 true 的话，BitmapFactory 将不会重复利用 Bitmap 内存，并输出相应 warning 日志：</p>
<blockquote>
<p>W/BitmapFactory: Unable to reuse an immutable bitmap as an image decoder target.</p>
</blockquote>
<p>完整的代码可以查看：<a href="https://github.com/McoyJiang/LagouAndroidShare/tree/master/course18_Bitmap/LagouBitmap">拉勾 AndroidBitmap</a></p>
<h3>BitmapRegionDecoder 图片分片显示</h3>
<p>有时候我们想要加载显示的图片很大或者很长，比如手机滚动截图功能生成的图片。</p>
<blockquote>
<p>针对这种情况，在不压缩图片的前提下，不建议一次性将整张图加载到内存，而是采用分片加载的方式来显示图片部分内容，然后根据手势操作，放大缩小或者移动图片显示区域。</p>
</blockquote>
<p>图片分片加载显示主要是使用 Android SDK 中的 BitmapRegionDecoder 来实现。用下面这张图rodman3.png&nbsp;举例：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AF/CgqCHl7GJdyANF1HAACsiTjhRw869.jpeg" alt="image.jpeg"></p>
<p><strong>BitmapRegionDecoder 基本使用</strong></p>
<p>首先需要使用 BitmapRegionDecoder 将图片加载到内存中，图片可以以绝对路径、文件描述符、输入流的方式传递给 BitmapRegionDecoder，如下所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A4/Ciqc1F7GJeeAFD6CAAFug--inhA254.png" alt="image (9).png"></p>
<p>运行后显示效果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A4/Ciqc1F7GJe2AX63nAAXfDN4qgKU233.png" alt="image (10).png"></p>
<p>在此基础上，我们可以通过自定义View，添加 touch 事件来动态地设置 Bitmap 需要显示的区域 Rect。具体实现网上已经有很多成熟的轮子可以直接使用，比如&nbsp;<a href="https://github.com/LuckyJayce/LargeImage/blob/master/library/src/main/java/com/shizhefei/view/largeimage/LargeImageView.java">LargeImageView</a>&nbsp;。张鸿洋先生也有一篇比较详细文章对此介绍：<a href="https://blog.csdn.net/lmj623565791/article/details/49300989">Android 高清加载巨图方案</a>。</p>
<h3>Bitmap 缓存</h3>
<p>当需要在界面上同时展示一大堆图片的时候，比如 ListView、RecyclerView 等，由于用户不断地上下滑动，某个 Bitmap 可能会被短时间内加载并销毁多次。这种情况下通过使用适当的缓存，可以有效地减缓 GC 频率保证图片加载效率，提高界面的响应速度和流畅性。</p>
<p>最常用的缓存方式就是 LruCache，基本使用方式如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A4/Ciqc1F7GJfaAG6-mAAU9UuZI15w565.png" alt="image (11).png"></p>
<p>解释说明：</p>
<ul>
<li>图中 1 处指定 LruCache 的最大空间为 20M，当超过 20M 时，LruCache 会根据内部缓存策略将多余 Bitmap 移除。</li>
<li>图中 2 处指定了插入 Bitmap 时的大小，当我们向 LruCache 中插入数据时，LruCache 并不知道每一个对象会占用大多内存，因此需要我们手动指定，并且根据缓存数据的类型不同也会有不同的计算方式。</li>
</ul>
<h3>总结：</h3>
<p>这节课详细介绍了 Bitmap 开发中的几个常见问题：</p>
<ol>
<li>一张图片被加载成 Bitmap 后实际占用内存是多大。</li>
<li>通过 Options.inBitmap 可以实现 Bitmap 的复用，但是有一定的限制。</li>
<li>当界面需要展示多张图片，尤其是在列表视图中，可以考虑使用 Bitmap 缓存。</li>
<li>如果需要展示的图片过大，可以考虑使用分片加载的策略</li>
</ol>

---

### 精选评论

##### **飞：
> 硬核！

##### **灿：
> 讲Android的好课太少了，难度始终，又扩展深度和广度，希望老师能多出几门

##### **昌：
> 日常追

##### **阳：
> 课代表来总结了：<div>bitmap 的优化策略</div><div>1.ARGB_888 改为 RGB_565 每个像素点 size/2</div><div>2.inSampleSize 长宽都间隔像素点采样 size/4</div><div>3.inBitmap 字段使 bitmap 复用</div><div>4.BitmapRegionDecoder 大图长图根据展示区域加载到内存，随着滑动手势来调整展示区域</div><div>5.LruCache 结构的内存缓存</div>

##### *鹏：
> 非常实用

##### **杰：
> 涨知识

##### *献：
> 不知不觉也看了一半了

##### **5260：
> 很实用的使用场景

##### **9169：
> 还一直在看，多谢分享

