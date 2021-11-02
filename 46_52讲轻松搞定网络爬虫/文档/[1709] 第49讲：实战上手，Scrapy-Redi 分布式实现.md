<p data-nodeid="13155">在前面一节课我们了解了 Scrapy-Redis 的基本原理，本节课我们就结合之前的案例实现基于 Scrapy-Redis 的分布式爬虫吧。</p>
<h3 data-nodeid="13156" class="">环境准备</h3>
<p data-nodeid="13157" class="">本节案例我们基于第 46 讲 —— Scrapy 和 Pyppeteer 的动态渲染页面的抓取案例来进行学习，我们需要把它改写成基于 Redis 的分布式爬虫。</p>



<p data-nodeid="13583" class="">首先我们需要把代码下载下来，其 GitHub 地址为 <a href="https://github.com/Python3WebSpider/ScrapyPyppeteer" data-nodeid="13587">https://github.com/Python3WebSpider/ScrapyPyppeteer</a>，进入项目，试着运行代码确保可以顺利执行，运行效果如图所示：<br>
<img src="https://s0.lgstatic.com/i/image/M00/3B/73/CgqCHl8kBHeAIw9uABBbpUkhs8c906.png" alt="1.png" data-nodeid="13592"><br>
其次，我们需要有一个 Redis 数据库，可以直接下载安装包并安装，也可以使用 Docker 启动，保证能正常连接和使用即可，比如我这里就在本地 localhost 启动了一个 Redis 数据库，运行在 6379 端口，密码为空。</p>



<p data-nodeid="13815">另外我们还需要安装 Scrapy-Redis 包，安装命令如下：</p>
<pre class="lang-java" data-nodeid="14763"><code data-language="java">pip3 install scrapy-redis
</code></pre>




<p data-nodeid="14982">安装完毕之后确保其可以正常导入使用即可。</p>
<h3 data-nodeid="15270" class="">实现</h3>
<p data-nodeid="15556">接下来我们只需要简单的几步操作就可以实现分布式爬虫的配置了。</p>
<h4 data-nodeid="15557" class="">修改 Scheduler</h4>
<p data-nodeid="15840" class="">在前面的课时中我们讲解了 Scheduler 的概念，它是用来处理 Request、Item 等对象的调度逻辑的，默认情况下，Request 的队列是在内存中的，为了实现分布式，我们需要将队列迁移到 Redis 中，这时候我们就需要修改 Scheduler，修改非常简单，只需要在 settings.py 里面添加如下代码即可：</p>
<pre data-nodeid="15841" class=""><code>SCHEDULER = "scrapy_redis.scheduler.Scheduler"
</code></pre>
<p data-nodeid="16102">这里我们将 Scheduler 的类修改为 Scrapy-Redis 提供的 Scheduler 类，这样在我们运行爬虫时，Request 队列就会出现在 Redis 中了。</p>
<h4 data-nodeid="16103" class="">修改 Redis 连接信息</h4>
<p data-nodeid="16361" class="">另外我们还需要修改下 Redis 的连接信息，这样 Scrapy 才能成功连接到 Redis 数据库，修改格式如下：</p>
<pre data-nodeid="16362" class=""><code>REDIS_URL = 'redis://[user:pass]@hostname:9001'
</code></pre>
<p data-nodeid="16586">在这里我们需要根据如上的格式来修改，由于我的 Redis 是在本地运行的，所以在这里就不需要填写用户名密码了，直接设置为如下内容即可：</p>
<pre data-nodeid="16587" class=""><code>REDIS_URL = 'redis://localhost:6379'
</code></pre>
<h4 data-nodeid="16790" class="">修改去重类</h4>
<p data-nodeid="16991">既然 Request 队列迁移到了 Redis，那么相应的去重操作我们也需要迁移到 Redis 里面，前一节课我们讲解了 Dupefilter 的原理，这里我们就修改下去重类来实现基于 Redis 的去重：</p>
<pre data-nodeid="16992" class=""><code>DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
</code></pre>
<h4 data-nodeid="17166" class="">配置持久化</h4>
<p data-nodeid="17338">一般来说开启了 Redis 分布式队列之后，我们不希望爬虫在关闭时将整个队列和去重信息全部删除，因为很有可能在某个情况下我们会手动关闭爬虫或者爬虫遭遇意外终止，为了解决这个问题，我们可以配置 Redis 队列的持久化，修改如下：</p>
<pre data-nodeid="17339" class=""><code>SCHEDULER_PERSIST = True
</code></pre>
<p data-nodeid="17499">好了，到此为止我们就完成分布式爬虫的配置了。</p>
<h3 data-nodeid="17500" class="">运行</h3>
<p data-nodeid="17501" class="">上面我们完成的实际上并不是真正意义的分布式爬虫，因为 Redis 队列我们使用的是本地的 Redis，所以多个爬虫需要运行在本地才可以，如果想实现真正意义的分布式爬虫，可以使用远程 Redis，这样我们就能在多台主机运行爬虫连接此 Redis 从而实现真正意义上的分布式爬虫了。</p>














<p data-nodeid="17657">不过没关系，我们可以在本地启动多个爬虫验证下爬取效果。我们在多个命令行窗口运行如下命令：</p>
<pre data-nodeid="17658" class=""><code>scrapy crawl book
</code></pre>
<p data-nodeid="17982">第一个爬虫出现了如下运行效果：<br>
<img src="https://s0.lgstatic.com/i/image/M00/3B/73/CgqCHl8kBSuAHjAvABs5G4LewHk110.png" alt="2.png" data-nodeid="17989"><br>
这时候不要关闭此窗口，再打开另外一个窗口，运行同样的爬取命令：</p>
<pre data-nodeid="17983" class=""><code>scrapy crawl book
</code></pre>
<p data-nodeid="18149" class="">运行效果如下：<br>
<img src="https://s0.lgstatic.com/i/image/M00/3B/68/Ciqc1F8kBUOAXh_TABoUyMu0pFo413.png" alt="3.png" data-nodeid="18154"><br>
这时候我们可以观察到它从第 24 页开始爬取了，因为当前爬取队列存在第一个爬虫生成的爬取 Request，第二个爬虫启动时检测到有 Request 存在就直接读取已经存在的 Request，然后接着爬取了。</p>






<p data-nodeid="12641">同样，我们可以启动第三个、第四个爬虫实现同样的爬取功能。这样，我们就基于 Scrapy-Redis 成功实现了基本的分布式爬虫功能。</p>
<p data-nodeid="12642" class="">好了，本课时的内容就讲完了，我们下节课见。</p>

---

### 精选评论


