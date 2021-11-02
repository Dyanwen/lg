<p>OkHttp 是一套处理 HTTP 网络请求的依赖库，由 Square 公司设计研发并开源，目前可以在 Java 和 Kotlin 中使用。对于 Android App 来说，OkHttp 现在几乎已经占据了所有的网络请求操作，RetroFit + OkHttp 实现网络请求似乎成了一种标配。因此它也是每一个 Android 开发工程师的必备技能，了解其内部实现原理可以更好地进行功能扩展、封装以及优化。</p>
<blockquote>
<p>因为是 HTTP 网络请求的依赖库，所以需要有一定的网络知识基础，这里我推荐一本入门级的书籍《网络是怎样连接的》（由日本作者户根勤所著）。这本书对入门级开发人员极为友好，能够快速帮你梳理从前端到服务器间的整套网络流程。</p>
</blockquote>
<h3>网络请求流程分析</h3>
<p>在 2016 年我写过几篇对 OkHttp 深入分析的文章：<a href="https://blog.csdn.net/zxm317122667/article/details/53202110">OkHttp的整个异步请求流程</a>。但是几年过去了，OkHttp 经过几次迭代后，已经发生了很多变化。更好的 WebSocket 支持、更多的 Interceptor 责任链，甚至连最核心的 HttpEngine 也变成了 HttpCodec。本课时会在最新的 3.10.0 的基础上，重新梳理整个网络请求的流程，以及实现机制。</p>
<p>先看下 OkHttp 的基本使用：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2B/Ciqc1F7Dk-eAItVSAADuEPXjYPo944.png" alt="image001.png"></p>
<p>除了直接 new OkHttpClient 之外，还可以使用内部工厂类 Builder 来设置 OkHttpClient。如下所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2B/Ciqc1F7Dk--AZk_0AADHHDXvjvc960.png" alt="image003.png"></p>
<p>请求操作的起点从 OkHttpClient.newCall().enqueue() 方法开始：</p>
<ul>
<li><strong>newCall</strong></li>
</ul>
<p>这个方法会返回一个 RealCall 类型的对象，通过它将网络请求操作添加到请求队列中。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2B/Ciqc1F7Dk_mAeO5DAABcf9D4khg568.png" alt="image005.png"></p>
<ul>
<li><strong>RealCall.enqueue</strong></li>
</ul>
<p>调用 Dispatcher 的入队方法，执行一个异步网络请求的操作。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2B/Ciqc1F7DlACAZ0ypAAD7tUY7Aps196.png" alt="image007.png"></p>
<p>可以看出，最终请求操作是委托给 Dispatcher的enqueue 方法内实现的。</p>
<blockquote>
<p>Dispatcher 是 OkHttpClient 的调度器，是一种门户模式。主要用来实现执行、取消异步请求操作。本质上是内部维护了一个线程池去执行异步操作，并且在 Dispatcher 内部根据一定的策略，保证最大并发个数、同一 host 主机允许执行请求的线程个数等。</p>
</blockquote>
<p>Dispatcher的enqueue 方法的具体实现如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2B/Ciqc1F7DlAeAY_UjAADrZmzFV74873.png" alt="image009.png"></p>
<p>可以看出，实际上就是使用线程池执行了一个 AsyncCall，而 AsyncCall 实现了 Runnable 接口，因此整个操作会在一个子线程（非 UI 线程）中执行。</p>
<p>继续查看 AsyncCall 中的 run 方法如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2B/Ciqc1F7DlA-AXkBNAAPgqEik5MY052.png" alt="image011.png"></p>
<p>在 run 方法中执行了另一个 execute 方法，而真正获取请求结果的方法是在 getResponseWithInterceptorChain 方法中，从名字也能看出其内部是一个拦截器的调用链，具体代码如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2B/Ciqc1F7DlBqAHUznAAJGJV3xQJg366.png" alt="image013.png"></p>
<p>每一个拦截器的作用如下。</p>
<ul>
<li>BridgeInterceptor：主要对 Request 中的 Head 设置默认值，比如 Content-Type、Keep-Alive、Cookie 等。</li>
<li>CacheInterceptor：负责 HTTP 请求的缓存处理。</li>
<li>ConnectInterceptor：负责建立与服务器地址之间的连接，也就是 TCP 链接。</li>
<li>CallServerInterceptor：负责向服务器发送请求，并从服务器拿到远端数据结果。</li>
</ul>
<blockquote>
<p>在添加上述几个拦截器之前，会调用 client.interceptors 将开发人员设置的拦截器添加到列表当中。</p>
</blockquote>
<p>对于 Request 的 Head 以及 TCP 链接，我们能控制修改的成分不是很多。所以本课时我重点讲解 CacheInterceptor 和 CallServerInterceptor。</p>
<h3>CacheInterceptor&nbsp;缓存拦截器</h3>
<h4>CacheInterceptor 主要做以下几件事情：</h4>
<p>a. 根据 Request 获取当前已有缓存的 Response（有可能为 null），并根据获取到的缓存 Response，创建 CacheStrategy 对象。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/37/CgqCHl7DlCOAdC6cAAEawroM0Uo610.png" alt="image015.png"></p>
<p>b. 通过 CacheStrategy 判断当前缓存中的 Response 是否有效（比如是否过期），如果缓存 Response 可用则直接返回，否则调用 chain.proceed() 继续执行下一个拦截器，也就是发送网络请求从服务器获取远端 Response。具体如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/37/CgqCHl7DlCuAWirnAAITtFPlLeA709.png" alt="image017.png"></p>
<p>c. 如果从服务器端成功获取 Response，再判断是否将此 Response 进行缓存操作。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/37/CgqCHl7DlDKAXXltAAG2Dz3ktBs683.png" alt="image019.png"></p>
<h4>通过 Cache 实现缓存功能</h4>
<p>通过上面分析缓存拦截器的流程可以看出，OkHttp 只是规范了一套缓存策略，但是具体使用何种方式将数据缓存到本地，以及如何从本地缓存中取出数据，都是由开发人员自己定义并实现，并通过 OkHttpClient.Builder 的 cache 方法设置。</p>
<p>OkHttp 提供了一个默认的缓存类 Cache.java，我们可以在构建 OkHttpClient 时，直接使用 Cache 来实现缓存功能。只需要指定缓存的路径，以及最大可用空间即可，如下所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/37/CgqCHl7DlDuAbKKmAACUEzX0A2k986.png" alt="image021.png"></p>
<p>上述代码使用 Android app 内置 cache 目录作为缓存路径，并设置缓存可用最大空间为 20M。实际上在 Cache 内部使用了 DiskLruCach 来实现具体的缓存功能，如下所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/37/CgqCHl7DlEKAfkCcAADcFHFhEIE069.png" alt="image023.png"></p>
<p>DiskLruCache 最终会以 journal 类型文件将需要缓存的数据保存在本地。如果感觉 OkHttp 自带的这套缓存策略太过复杂，我们可以设置使用 DiskLruCache 自己实现缓存机制。</p>
<h4>CallServerInterceptor 详解</h4>
<p>CallServerInterceptor 是 OkHttp 中最后一个拦截器，也是 OkHttp 中最核心的网路请求部分，其 intercept 方法如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/37/CgqCHl7DlEyAAYQ5AANbdjxrvDk061.png" alt="image025.png"></p>
<p>如上图所示，主要分为 2 部分。蓝线以上的操作是向服务器端发送请求数据，蓝线以下代表从服务端获取相应数据并构建 Response 对象。</p>
<h4>OkHttp 使用扩展</h4>
<p>仔细看刚才 CallServerInterceptor 中的 intercept 方法，可以发现在向服务端发送数据以及获取数据都是使用一个 Okio 的框架来实现的。Okio 是 Square 公司打造的另外一个轻量级 IO 库，它是 OkHttp 框架的基石。</p>
<p>我们在构建 Response 时，需要调动 body() 方法传入一个 ResponseBody 对象。ResponseBody 内部封装了对请求结果的流读取操作。我们可以通过继承并扩展 ResponseBody 的方式获取网络请求的进度。</p>
<p><strong>a.</strong> 继承 ResponseBody</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/38/CgqCHl7DlFiAfsCnAANPz4157cU538.png" alt="image027.png"></p>
<p>其中 progressListener 是一个自定义的进度监听器，通过它向上层汇报网络请求的进度。</p>
<p><strong>b.</strong> 自定义 ProgressBarClient</p>
<p>代码如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2C/Ciqc1F7DlGCADNj3AAJmGXwuBs4793.png" alt="image029.png"></p>
<p>解释说明：</p>
<p>getClient 可以根据项目的不同添加其他共通设置，比如 timeout 时间、DNS、Log 日志 interceptor 等。</p>
<p>getProgressBarClient 通过添加一个拦截器，并且在 intercept 方法中将自定义的 ProgressResponseBody 传给 body 方法。</p>
<p>当通过 getProgressBarClient 发送网络请求时，OkHttpClient 从服务端获取到数据之后，会不断调用 ProgressResponseBody 中的 source 方法，然后通过 ProgressListener 向上层通知请求进度的结果。</p>
<p><strong>c.</strong>&nbsp;实践拓展 — Picasso</p>
<p>我们甚至可以将上面自定义的 ProgressBarClient 用在 Square 公司另外一个请求库—Picasso。Picasso 是 Square 公司研发用来从网络端获取图片数据的依赖库，内部实质上是使用 OkHttp 来实现请求操作的。因此我们可以将 ProgressBarClient 替换 Picasso 中的 OkHttpClient，这样就能获取下载图片的进度，代码如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2C/Ciqc1F7DlGqAAo5gAAD01zTM0Cg633.png" alt="image031.png"></p>
<p>后续只要通过 getPicasso 方法即可获得一个自带下载进度的 Picasso 对象，因为 OkHttp、Picasso、Okio 都来自 Square 公司，所以我将这些工具栏的 get 方法放在一个 SquareUtils 类中，我已经上传到代码仓库中，详细代码可以查看：<a href="https://github.com/McoyJiang/LagouAndroidShare/blob/master/course17_OkHttp/SquareUtils.java">SquareUtils.java</a></p>
<p>如果结合第 15 课时用过的 PieImageView，我们就可以实现一个带进度提示的图片下载效果，代码如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/38/CgqCHl7DlHKAWCgpAAECSW93DuE778.png" alt="image033.png"></p>
<p>最终运行效果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/2C/Ciqc1F7DlH-AKnYgACIcXjP9IHk410.gif" alt="image035.gif"></p>
<h3>总结</h3>
<p>这节课主要分析了 OkHttp 的源码实现：</p>
<ul>
<li>首先 OkHttp 内部是一个门户模式，所有的下发工作都是通过一个门户 Dispatcher 来进行分发。</li>
<li>然后在网络请求阶段通过责任链模式，链式的调用各个拦截器的 intercept 方法。其中我重点介绍了 2 个比较重要的拦截器：CacheInterceptor 和 CallServerInterceptor。它们分别用来做请求缓存和执行网络请求操作。</li>
<li>最后我们在理解源码实现的基础上，对 OkHttp 的功能进行了一些扩展，实现了网络请求进度的实现。</li>
</ul>

---

### 精选评论

##### **0102：
> 收获颇丰，感谢老师😀

##### **服务员：
> 来了,来了,他来了

##### **好：
> <span style="font-size: 15.372px;">门户模式是指门面模式吗</span>

