<p data-nodeid="1449" class="">在前面的示例中我们已经了解了 Item Pipeline 项目管道的基本概念，本节课我们就深入详细讲解它的用法。</p>
<p data-nodeid="1450">首先我们看看 Item Pipeline 在 Scrapy 中的架构，如图所示。</p>
<p data-nodeid="1451"><img src="https://s0.lgstatic.com/i/image/M00/32/C9/Ciqc1F8OyNqAAbnKAAJygBiwVD4320.png" alt="Drawing 0.png" data-nodeid="1597"></p>
<p data-nodeid="1452">图中的最左侧即为 Item Pipeline，它的调用发生在 Spider 产生 Item 之后。当 Spider 解析完 Response 之后，Item 就会传递到 Item Pipeline，被定义的 Item Pipeline 组件会顺次调用，完成一连串的处理过程，比如数据清洗、存储等。</p>
<p data-nodeid="1453">它的主要功能有：</p>
<ul data-nodeid="1454">
<li data-nodeid="1455">
<p data-nodeid="1456">清洗 HTML 数据；</p>
</li>
<li data-nodeid="1457">
<p data-nodeid="1458">验证爬取数据，检查爬取字段；</p>
</li>
<li data-nodeid="1459">
<p data-nodeid="1460">查重并丢弃重复内容；</p>
</li>
<li data-nodeid="1461">
<p data-nodeid="1462">将爬取结果储存到数据库。</p>
</li>
</ul>
<h3 data-nodeid="1463">1. 核心方法</h3>
<p data-nodeid="1464">我们可以自定义 Item Pipeline，只需要实现指定的方法就可以，其中必须要实现的一个方法是：</p>
<ul data-nodeid="1465">
<li data-nodeid="1466">
<p data-nodeid="1467">process_item(item, spider)</p>
</li>
</ul>
<p data-nodeid="1468">另外还有几个比较实用的方法，它们分别是：</p>
<ul data-nodeid="1469">
<li data-nodeid="1470">
<p data-nodeid="1471">open_spider(spider)</p>
</li>
<li data-nodeid="1472">
<p data-nodeid="1473">close_spider(spider)</p>
</li>
<li data-nodeid="1474">
<p data-nodeid="1475">from_crawler(cls, crawler)</p>
</li>
</ul>
<p data-nodeid="1476">下面我们对这几个方法的用法做下详细的介绍：</p>
<h4 data-nodeid="1477">process_item(item, spider)</h4>
<p data-nodeid="1478">process_item() 是必须要实现的方法，被定义的 Item Pipeline 会默认调用这个方法对 Item 进行处理。比如，我们可以进行数据处理或者将数据写入数据库等操作。它必须返回 Item 类型的值或者抛出一个 DropItem 异常。</p>
<p data-nodeid="1479">process_item() 方法的参数有如下两个：</p>
<ul data-nodeid="1480">
<li data-nodeid="1481">
<p data-nodeid="1482">item，是 Item 对象，即被处理的 Item；</p>
</li>
<li data-nodeid="1483">
<p data-nodeid="1484">spider，是 Spider 对象，即生成该 Item 的 Spider。</p>
</li>
</ul>
<p data-nodeid="1485">下面对该方法的返回类型归纳如下：</p>
<ul data-nodeid="1486">
<li data-nodeid="1487">
<p data-nodeid="1488">如果返回的是 Item 对象，那么此 Item 会被低优先级的 Item Pipeline 的 process_item() 方法进行处理，直到所有的方法被调用完毕。</p>
</li>
<li data-nodeid="1489">
<p data-nodeid="1490">如果抛出的是 DropItem 异常，那么此 Item 就会被丢弃，不再进行处理。</p>
</li>
</ul>
<h4 data-nodeid="1491">open_spider(self, spider)</h4>
<p data-nodeid="1492">open_spider() 方法是在 Spider 开启的时候被自动调用的，在这里我们可以做一些初始化操作，如开启数据库连接等。其中参数 spider 就是被开启的 Spider 对象。</p>
<h4 data-nodeid="1493">close_spider(spider)</h4>
<p data-nodeid="1494">close_spider() 方法是在 Spider 关闭的时候自动调用的，在这里我们可以做一些收尾工作，如关闭数据库连接等，其中参数 spider 就是被关闭的 Spider 对象。</p>
<h4 data-nodeid="1495">from_crawler(cls, crawler)</h4>
<p data-nodeid="1496">from_crawler() 方法是一个类方法，用 @classmethod 标识，是一种依赖注入的方式。它的参数是 crawler，通过 crawler 对象，我们可以拿到 Scrapy 的所有核心组件，如全局配置的每个信息，然后创建一个 Pipeline 实例。参数 cls 就是 Class，最后返回一个 Class 实例。</p>
<p data-nodeid="1497">下面我们用一个实例来加深对 Item Pipeline 用法的理解。</p>
<h3 data-nodeid="1498">2. 本节目标</h3>
<p data-nodeid="1499">我们以爬取 360 摄影美图为例，来分别实现 MongoDB 存储、MySQL 存储、Image 图片存储的三个 Pipeline。</p>
<h3 data-nodeid="1500">3. 准备工作</h3>
<p data-nodeid="1501">请确保已经安装好 MongoDB 和 MySQL 数据库，安装好 Python 的 PyMongo、PyMySQL、Scrapy 框架，另外需要安装 pillow 图像处理库，如果没有安装可以参考前文的安装说明。</p>
<h3 data-nodeid="1502">4. 抓取分析</h3>
<p data-nodeid="1503">我们这次爬取的目标网站为：<a href="https://image.so.com" data-nodeid="1671">https://image.so.com</a>。打开此页面，切换到摄影页面，网页中呈现了许许多多的摄影美图。我们打开浏览器开发者工具，过滤器切换到 XHR 选项，然后下拉页面，可以看到下面就会呈现许多 Ajax 请求，如图所示。</p>
<p data-nodeid="1504"><img src="https://s0.lgstatic.com/i/image/M00/32/D4/CgqCHl8OyPCATwEaAA4FKjXCH38464.png" alt="Drawing 1.png" data-nodeid="1675"></p>
<p data-nodeid="1505">我们查看一个请求的详情，观察返回的数据结构，如图所示。</p>
<p data-nodeid="1506"><img src="https://s0.lgstatic.com/i/image/M00/32/C9/Ciqc1F8OyPiAHQO_AAKWgKacbAY613.png" alt="Drawing 2.png" data-nodeid="1679"></p>
<p data-nodeid="1507">返回格式是 JSON。其中 list 字段就是一张张图片的详情信息，包含了 30 张图片的 ID、名称、链接、缩略图等信息。另外观察 Ajax 请求的参数信息，有一个参数 sn 一直在变化，这个参数很明显就是偏移量。当 sn 为 30 时，返回的是前 30 张图片，sn 为 60 时，返回的就是第 31~60 张图片。另外，ch 参数是摄影类别，listtype 是排序方式，temp 参数可以忽略。</p>
<p data-nodeid="1508">所以我们抓取时只需要改变 sn 的数值就好了。下面我们用 Scrapy 来实现图片的抓取，将图片的信息保存到 MongoDB、MySQL，同时将图片存储到本地。</p>
<h3 data-nodeid="1509">5. 新建项目</h3>
<p data-nodeid="1510">首先新建一个项目，命令如下：</p>
<pre class="lang-java" data-nodeid="1511"><code data-language="java">scrapy startproject images360
</code></pre>
<p data-nodeid="1512">接下来新建一个 Spider，命令如下：</p>
<pre class="lang-java" data-nodeid="1513"><code data-language="java">scrapy genspider images images.so.com
</code></pre>
<p data-nodeid="1514">这样我们就成功创建了一个 Spider。</p>
<h3 data-nodeid="1515">6. 构造请求</h3>
<p data-nodeid="1516">接下来定义爬取的页数。比如爬取 50 页、每页 30 张，也就是 1500 张图片，我们可以先在 settings.py 里面定义一个变量 MAX_PAGE，添加如下定义：</p>
<pre class="lang-java" data-nodeid="1517"><code data-language="java">MAX_PAGE = <span class="hljs-number">50</span>
</code></pre>
<p data-nodeid="1518">定义 start_requests() 方法，用来生成 50 次请求，如下所示：</p>
<pre class="lang-java" data-nodeid="1519"><code data-language="java"><span class="hljs-function">def <span class="hljs-title">start_requests</span><span class="hljs-params">(self)</span>:
 &nbsp; &nbsp;data </span>= {<span class="hljs-string">'ch'</span>: <span class="hljs-string">'photography'</span>, <span class="hljs-string">'listtype'</span>: <span class="hljs-string">'new'</span>}
 &nbsp; &nbsp;base_url = <span class="hljs-string">'https://image.so.com/zjl?'</span>
 &nbsp; &nbsp;<span class="hljs-function"><span class="hljs-keyword">for</span> page in <span class="hljs-title">range</span><span class="hljs-params">(<span class="hljs-number">1</span>, self.settings.get(<span class="hljs-string">'MAX_PAGE'</span>)</span> + 1):
 &nbsp; &nbsp; &nbsp; &nbsp;data['sn'] </span>= page * <span class="hljs-number">30</span>
 &nbsp; &nbsp; &nbsp; &nbsp;params = urlencode(data)
 &nbsp; &nbsp; &nbsp; &nbsp;url = base_url + <span class="hljs-function">params
 &nbsp; &nbsp; &nbsp; &nbsp;yield <span class="hljs-title">Request</span><span class="hljs-params">(url, self.parse)</span>
</span></code></pre>
<p data-nodeid="1520">在这里我们首先定义了初始的两个参数，sn 参数是遍历循环生成的。然后利用 urlencode 方法将字典转化为 URL 的 GET 参数，构造出完整的 URL，构造并生成 Request。</p>
<p data-nodeid="1521">还需要引入 scrapy.Request 和 urllib.parse 模块，如下所示：</p>
<pre class="lang-java" data-nodeid="1522"><code data-language="java">from scrapy <span class="hljs-keyword">import</span> Spider, Request
from urllib.parse <span class="hljs-keyword">import</span> urlencode
</code></pre>
<p data-nodeid="1523">再修改 settings.py 中的 ROBOTSTXT_OBEY 变量，将其设置为 False，否则无法抓取，如下所示：</p>
<pre class="lang-java" data-nodeid="1524"><code data-language="java">ROBOTSTXT_OBEY = False
</code></pre>
<p data-nodeid="1525">运行爬虫，即可以看到链接都请求成功，执行命令如下所示：</p>
<pre class="lang-java" data-nodeid="1526"><code data-language="java">scrapy crawl images
</code></pre>
<p data-nodeid="1527">运行示例结果如图所示。</p>
<p data-nodeid="1528"><img src="https://s0.lgstatic.com/i/image/M00/32/C9/Ciqc1F8OyReAWgKjAAGstwAqxoU157.png" alt="Drawing 3.png" data-nodeid="1708"></p>
<p data-nodeid="1529">所有请求的状态码都是 200，这就证明图片信息爬取成功了。</p>
<h3 data-nodeid="1530">7. 提取信息</h3>
<p data-nodeid="1531">首先定义一个叫作 ImageItem 的 Item，如下所示：</p>
<pre class="lang-java" data-nodeid="1532"><code data-language="java">from scrapy <span class="hljs-keyword">import</span> Item, <span class="hljs-function">Field
class <span class="hljs-title">ImageItem</span><span class="hljs-params">(Item)</span>:
 &nbsp; &nbsp;collection </span>= table = <span class="hljs-string">'images'</span>
 &nbsp; &nbsp;id = Field()
 &nbsp; &nbsp;url = Field()
 &nbsp; &nbsp;title = Field()
 &nbsp; &nbsp;thumb = Field()
</code></pre>
<p data-nodeid="1533">在这里我们定义了 4 个字段，包括图片的 ID、链接、标题、缩略图。另外还有两个属性 collection 和 table，都定义为 images 字符串，分别代表 MongoDB 存储的 Collection 名称和 MySQL 存储的表名称。<br>
接下来我们提取 Spider 里有关信息，将 parse 方法改写为如下所示：</p>
<pre class="lang-java" data-nodeid="1534"><code data-language="java"><span class="hljs-function">def <span class="hljs-title">parse</span><span class="hljs-params">(self, response)</span>:
    result </span>= json.loads(response.text)
    <span class="hljs-keyword">for</span> image in result.get(<span class="hljs-string">'list'</span>):
        item = ImageItem()
        item[<span class="hljs-string">'id'</span>] = image.get(<span class="hljs-string">'id'</span>)
        item[<span class="hljs-string">'url'</span>] = image.get(<span class="hljs-string">'qhimg_url'</span>)
        item[<span class="hljs-string">'title'</span>] = image.get(<span class="hljs-string">'title'</span>)
        item[<span class="hljs-string">'thumb'</span>] = image.get(<span class="hljs-string">'qhimg_thumb'</span>)
        yield item
</code></pre>
<p data-nodeid="1535">首先解析 JSON，遍历其 list 字段，取出一个个图片信息，然后再对 ImageItem 进行赋值，生成 Item 对象。<br>
这样我们就完成了信息的提取。</p>
<h3 data-nodeid="1536">8. 存储信息</h3>
<p data-nodeid="1537">接下来我们需要将图片的信息保存到 MongoDB、MySQL 中，同时将图片保存到本地。</p>
<h4 data-nodeid="1538">MongoDB</h4>
<p data-nodeid="1539">首先确保 MongoDB 已经正常安装并且能够正常运行。</p>
<p data-nodeid="1540">我们用一个 MongoPipeline 将信息保存到 MongoDB 中，在 pipelines.py 里添加如下类的实现：</p>
<pre class="lang-java" data-nodeid="1541"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">import</span> pymongo
class <span class="hljs-title">MongoPipeline</span><span class="hljs-params">(object)</span>:
    def <span class="hljs-title">__init__</span><span class="hljs-params">(self, mongo_uri, mongo_db)</span>:
        self.mongo_uri </span>= mongo_uri
        self.mongo_db = mongo_db

    <span class="hljs-meta">@classmethod</span>
    <span class="hljs-function">def <span class="hljs-title">from_crawler</span><span class="hljs-params">(cls, crawler)</span>:
        return <span class="hljs-title">cls</span><span class="hljs-params">(mongo_uri=crawler.settings.get(<span class="hljs-string">'MONGO_URI'</span>)</span>,
            mongo_db</span>=crawler.settings.get(<span class="hljs-string">'MONGO_DB'</span>)
        )
    <span class="hljs-function">def <span class="hljs-title">open_spider</span><span class="hljs-params">(self, spider)</span>:
        self.client </span>= pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]
    <span class="hljs-function">def <span class="hljs-title">process_item</span><span class="hljs-params">(self, item, spider)</span>:
        self.db[item.collection].<span class="hljs-title">insert</span><span class="hljs-params">(dict(item)</span>)
        return item
    def <span class="hljs-title">close_spider</span><span class="hljs-params">(self, spider)</span>:
        self.client.<span class="hljs-title">close</span><span class="hljs-params">()</span>
</span></code></pre>
<p data-nodeid="1542">这里需要用到两个变量，MONGO_URI 和 MONGO_DB，即存储到 MongoDB 的链接地址和数据库名称。我们在 settings.py 里添加这两个变量，如下所示：</p>
<pre class="lang-java" data-nodeid="1543"><code data-language="java">MONGO_URI = <span class="hljs-string">'localhost'</span>
MONGO_DB = <span class="hljs-string">'images360'</span>
</code></pre>
<p data-nodeid="1544">这样一个保存到 MongoDB 的 Pipeline 的就创建好了。这里最主要的方法是 process_item()，直接调用 Collection 对象的 insert 方法即可完成数据的插入，最后返回 Item 对象。</p>
<h4 data-nodeid="1545">MySQL</h4>
<p data-nodeid="1546">首先需要确保 MySQL 已经正确安装并且正常运行。</p>
<p data-nodeid="1547">新建一个数据库，名字还是 images360，SQL 语句如下所示：</p>
<pre class="lang-java" data-nodeid="1548"><code data-language="java">CREATE DATABASE images360 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci
</code></pre>
<p data-nodeid="1549">新建一个数据表，包含 id、url、title、thumb 四个字段，SQL 语句如下所示：</p>
<pre class="lang-java" data-nodeid="1550"><code data-language="java"><span class="hljs-function">CREATE TABLE <span class="hljs-title">images</span> <span class="hljs-params">(id VARCHAR(<span class="hljs-number">255</span>)</span> NULL PRIMARY KEY, url <span class="hljs-title">VARCHAR</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span> NULL , title <span class="hljs-title">VARCHAR</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span> NULL , thumb <span class="hljs-title">VARCHAR</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span> NULL)
</span></code></pre>
<p data-nodeid="1551">执行完 SQL 语句之后，我们就成功创建好了数据表。接下来就可以往表里存储数据了。</p>
<p data-nodeid="1552">接下来我们实现一个 MySQLPipeline，代码如下所示：</p>
<pre class="lang-java" data-nodeid="1553"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">import</span> pymysql
class <span class="hljs-title">MysqlPipeline</span><span class="hljs-params">()</span>:
    def <span class="hljs-title">__init__</span><span class="hljs-params">(self, host, database, user, password, port)</span>:
        self.host </span>= host
        self.database = database
        self.user = user
        self.password = password
        self.port = port

    <span class="hljs-meta">@classmethod</span>
    <span class="hljs-function">def <span class="hljs-title">from_crawler</span><span class="hljs-params">(cls, crawler)</span>:
        return <span class="hljs-title">cls</span><span class="hljs-params">(host=crawler.settings.get(<span class="hljs-string">'MYSQL_HOST'</span>)</span>,
            database</span>=crawler.settings.get(<span class="hljs-string">'MYSQL_DATABASE'</span>),
            user=crawler.settings.get(<span class="hljs-string">'MYSQL_USER'</span>),
            password=crawler.settings.get(<span class="hljs-string">'MYSQL_PASSWORD'</span>),
            port=crawler.settings.get(<span class="hljs-string">'MYSQL_PORT'</span>),
        )

    <span class="hljs-function">def <span class="hljs-title">open_spider</span><span class="hljs-params">(self, spider)</span>:
        self.db </span>= pymysql.connect(self.host, self.user, self.password, self.database, charset=<span class="hljs-string">'utf8'</span>, port=self.port)
        self.cursor = self.db.cursor()

    <span class="hljs-function">def <span class="hljs-title">close_spider</span><span class="hljs-params">(self, spider)</span>:
        self.db.<span class="hljs-title">close</span><span class="hljs-params">()</span>

    def <span class="hljs-title">process_item</span><span class="hljs-params">(self, item, spider)</span>:
        data </span>= dict(item)
        keys = <span class="hljs-string">', '</span>.join(data.keys())
        values = <span class="hljs-string">', '</span>.join([<span class="hljs-string">'% s'</span>] * len(data))
        sql = <span class="hljs-string">'insert into % s (% s) values (% s)'</span> % (item.table, keys, values)
        self.cursor.execute(sql, tuple(data.values()))
        self.db.commit()
        <span class="hljs-keyword">return</span> item
</code></pre>
<p data-nodeid="1554">如前所述，这里用到的数据插入方法是一个动态构造 SQL 语句的方法。</p>
<p data-nodeid="1555">这里还需要几个 MySQL 的配置，我们在 settings.py 里添加几个变量，如下所示：</p>
<pre class="lang-java" data-nodeid="1556"><code data-language="java">MYSQL_HOST = <span class="hljs-string">'localhost'</span>
MYSQL_DATABASE = <span class="hljs-string">'images360'</span>
MYSQL_PORT = <span class="hljs-number">3306</span>
MYSQL_USER = <span class="hljs-string">'root'</span>
MYSQL_PASSWORD = <span class="hljs-string">'123456'</span>
</code></pre>
<p data-nodeid="1557">这里分别定义了 MySQL 的地址、数据库名称、端口、用户名、密码。这样，MySQL Pipeline 就完成了。</p>
<h4 data-nodeid="1558">Image Pipeline</h4>
<p data-nodeid="1559">Scrapy 提供了专门处理下载的 Pipeline，包括文件下载和图片下载。下载文件和图片的原理与抓取页面的原理一样，因此下载过程支持异步和多线程，十分高效。下面我们来看看具体的实现过程。</p>
<p data-nodeid="1560">官方文档地址为：<a href="https://doc.scrapy.org/en/latest/topics/media-pipeline.html" data-nodeid="1749">https://doc.scrapy.org/en/latest/topics/media-pipeline.html</a>。</p>
<p data-nodeid="1561">首先定义存储文件的路径，需要定义一个 IMAGES_STORE 变量，在 settings.py 中添加如下代码：</p>
<pre class="lang-java" data-nodeid="1562"><code data-language="java">IMAGES_STORE = <span class="hljs-string">'./images'</span>
</code></pre>
<p data-nodeid="1563">在这里我们将路径定义为当前路径下的 images 子文件夹，即下载的图片都会保存到本项目的 images 文件夹中。</p>
<p data-nodeid="1564">内置的 ImagesPipeline 会默认读取 Item 的 image_urls 字段，并认为该字段是一个列表形式，它会遍历 Item 的 image_urls 字段，然后取出每个 URL 进行图片下载。</p>
<p data-nodeid="1565">但是现在生成的 Item 的图片链接字段并不是 image_urls 字段表示的，也不是列表形式，而是单个的 URL。所以为了实现下载，我们需要重新定义下载的部分逻辑，即需要自定义 ImagePipeline，继承内置的 ImagesPipeline，重写方法。</p>
<p data-nodeid="1566">我们定义 ImagePipeline，如下所示：</p>
<pre class="lang-java" data-nodeid="1567"><code data-language="java">from scrapy <span class="hljs-keyword">import</span> Request
from scrapy.exceptions <span class="hljs-keyword">import</span> DropItem
from scrapy.pipelines.<span class="hljs-function">images <span class="hljs-keyword">import</span> ImagesPipeline
class <span class="hljs-title">ImagePipeline</span><span class="hljs-params">(ImagesPipeline)</span>:
    def <span class="hljs-title">file_path</span><span class="hljs-params">(self, request, response=None, info=None)</span>:
        url </span>= request.url
        file_name = url.split(<span class="hljs-string">'/'</span>)[-<span class="hljs-number">1</span>]
        <span class="hljs-keyword">return</span> <span class="hljs-function">file_name

    def <span class="hljs-title">item_completed</span><span class="hljs-params">(self, results, item, info)</span>:
        image_paths </span>= [x[<span class="hljs-string">'path'</span>] <span class="hljs-keyword">for</span> ok, x in results <span class="hljs-keyword">if</span> ok]
        <span class="hljs-keyword">if</span> not image_paths:
            <span class="hljs-function">raise <span class="hljs-title">DropItem</span><span class="hljs-params">(<span class="hljs-string">'Image Downloaded Failed'</span>)</span>
        return item

    def <span class="hljs-title">get_media_requests</span><span class="hljs-params">(self, item, info)</span>:
        yield <span class="hljs-title">Request</span><span class="hljs-params">(item[<span class="hljs-string">'url'</span>])</span>
</span></code></pre>
<p data-nodeid="1568">在这里我们实现了 ImagePipeline，继承 Scrapy 内置的 ImagesPipeline，重写了下面几个方法。</p>
<ul data-nodeid="1569">
<li data-nodeid="1570">
<p data-nodeid="1571">get_media_requests()。它的第一个参数 item 是爬取生成的 Item 对象。我们将它的 url 字段取出来，然后直接生成 Request 对象。此 Request 加入调度队列，等待被调度，执行下载。</p>
</li>
<li data-nodeid="1572">
<p data-nodeid="1573">file_path()。它的第一个参数 request 就是当前下载对应的 Request 对象。这个方法用来返回保存的文件名，直接将图片链接的最后一部分当作文件名即可。它利用 split() 函数分割链接并提取最后一部分，返回结果。这样此图片下载之后保存的名称就是该函数返回的文件名。</p>
</li>
<li data-nodeid="1574">
<p data-nodeid="1575">item_completed()，它是当单个 Item 完成下载时的处理方法。因为并不是每张图片都会下载成功，所以我们需要分析下载结果并剔除下载失败的图片。如果某张图片下载失败，那么我们就不需保存此 Item 到数据库。该方法的第一个参数 results 就是该 Item 对应的下载结果，它是一个列表形式，列表每一个元素是一个元组，其中包含了下载成功或失败的信息。这里我们遍历下载结果找出所有成功的下载列表。如果列表为空，那么该 Item 对应的图片下载失败，随即抛出异常 DropItem，该 Item 忽略。否则返回该 Item，说明此 Item 有效。</p>
</li>
</ul>
<p data-nodeid="1576">现在为止，三个 Item Pipeline 的定义就完成了。最后只需要启用就可以了，修改 settings.py，设置 ITEM_PIPELINES，如下所示：</p>
<pre class="lang-java" data-nodeid="1577"><code data-language="java">ITEM_PIPELINES = {
    <span class="hljs-string">'images360.pipelines.ImagePipeline'</span>: <span class="hljs-number">300</span>,
    <span class="hljs-string">'images360.pipelines.MongoPipeline'</span>: <span class="hljs-number">301</span>,
    <span class="hljs-string">'images360.pipelines.MysqlPipeline'</span>: <span class="hljs-number">302</span>,
}
</code></pre>
<p data-nodeid="1578">这里注意调用的顺序。我们需要优先调用 ImagePipeline 对 Item 做下载后的筛选，下载失败的 Item 就直接忽略，它们就不会保存到 MongoDB 和 MySQL 里。随后再调用其他两个存储的 Pipeline，这样就能确保存入数据库的图片都是下载成功的。<br>
接下来运行程序，执行爬取，如下所示：</p>
<pre class="lang-java" data-nodeid="1579"><code data-language="java">scrapy crawl images
</code></pre>
<p data-nodeid="1580">爬虫一边爬取一边下载，下载速度非常快，对应的输出日志如图所示。</p>
<p data-nodeid="1581"><img src="https://s0.lgstatic.com/i/image/M00/32/D5/CgqCHl8OyV2AWLyNAAGALb5Nqd8706.png" alt="Drawing 4.png" data-nodeid="1785"></p>
<p data-nodeid="1582">查看本地 images 文件夹，发现图片都已经成功下载，如图所示。</p>
<p data-nodeid="1583"><img src="https://s0.lgstatic.com/i/image/M00/32/D5/CgqCHl8OyWWAY76eAAKKY0ouxBs666.png" alt="Drawing 5.png" data-nodeid="1789"></p>
<p data-nodeid="1584">查看 MySQL，下载成功的图片信息也已成功保存，如图所示。</p>
<p data-nodeid="1585"><img src="https://s0.lgstatic.com/i/image/M00/32/CA/Ciqc1F8OyXaAdJc7ACY7Z4cSzuE317.png" alt="Drawing 6.png" data-nodeid="1793"></p>
<p data-nodeid="1586">查看 MongoDB，下载成功的图片信息同样已成功保存，如图所示。</p>
<p data-nodeid="1587"><img src="https://s0.lgstatic.com/i/image/M00/32/CA/Ciqc1F8OydSAD3T6ABH5qD8uKgM923.png" alt="Drawing 7.png" data-nodeid="1797"></p>
<p data-nodeid="1588">这样我们就可以成功实现图片的下载并把图片的信息存入数据库了。</p>
<h3 data-nodeid="1589">9. 本节代码</h3>
<p data-nodeid="1811" class="te-preview-highlight">本节代码地址为：<br>
<a href="https://github.com/Python3WebSpider/Images360" data-nodeid="1816">https://github.com/Python3WebSpider/Images360</a>。</p>

<h3 data-nodeid="1591">10. 结语</h3>
<p data-nodeid="1592" class="">Item Pipeline 是 Scrapy 非常重要的组件，数据存储几乎都是通过此组件实现的。请你务必认真掌握此内容。</p>

---

### 精选评论

##### *丹：
> 结合官方文档反复看了scrapy这个框架，才明白这个框架的精妙之处，在未看老师的课程之前，一直都是自己写全过程，现在可以改用scrapy。整体来看，scrapy 引擎是一个总指挥，负责协调各方资源，手下有Spider、Downloader、ItemPipeline以及Schedule,各司其职，另外分别提供了SpiderMiddleware和DownloaderMiddleware，可以更方面我们对Request、Response、Item进行进一步的处理，可以这么说，Spider进进出出的对象【或理解成数据】，会经过SpiderMiddleware，Downloader进进出出的对象，会经过Downloader Middleware

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 哇哦，小编给你点赞

