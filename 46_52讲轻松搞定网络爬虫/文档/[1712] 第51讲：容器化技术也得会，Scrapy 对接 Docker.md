<p data-nodeid="211175">上一节课我们学习了 Scrapy 和 Scrapyd 的用法，虽然它们可以解决项目部署的一些问题，但其实这种方案并没有真正彻底解决环境配置的问题。</p>
<p data-nodeid="211176">比如使用 Scrapyd 时我们依然需要安装对应的依赖库，即使这样仍免不了还是会出现环境冲突和不一致的问题。因此，本节课我会再介绍另一种部署方案 —— Docker。</p>
<p data-nodeid="211177">Docker 可以提供操作系统级别的虚拟环境，一个 Docker 镜像一般都会包含一个完整的操作系统，而这些系统内也有已经配置好的开发环境，如 Python 3.6 环境等。</p>
<p data-nodeid="211178">我们可以直接使用此 Docker 的 Python 3 镜像运行一个容器，将项目直接放到容器里运行，就不用再额外配置 Python 3 环境了，这样就解决了环境配置的问题。</p>
<p data-nodeid="211179">我们也可以进一步将 Scrapy 项目制作成一个新的 Docker 镜像，镜像里只包含适用于本项目的 Python 环境。如果要部署到其他平台，只需要下载该镜像并运行就好了，因为 Docker 运行时采用虚拟环境，和宿主机是完全隔离的，所以也不需要担心环境冲突问题。</p>
<p data-nodeid="211180">如果我们能够把 Scrapy 项目制作成一个 Docker 镜像，只要其他主机安装了 Docker，那么只要将镜像下载并运行即可，而不必再担心环境配置问题或版本冲突问题。</p>
<p data-nodeid="211181">因此，利用 Docker 我们就能很好地解决环境配置、环境冲突等问题。接下来，我们就尝试把一个 Scrapy 项目制作成一个 Docker 镜像。</p>
<h3 data-nodeid="212405" class="">本节目标</h3>


<p data-nodeid="211183">我们要实现把前文 Scrapy 的入门项目打包成一个 Docker 镜像的过程。项目爬取的网址为： <a href="http://quotes.toscrape.com/" data-nodeid="211263">http://quotes.toscrape.com/</a>，本模块 Scrapy 入门一节已经实现了 Scrapy 对此站点的爬取过程，项目代码为： <a href="https://github.com/Python3WebSpider/ScrapyTutorial" data-nodeid="211267">https://github.com/Python3WebSpider/ScrapyTutorial</a>，如果本地不存在的话可以 Clone 下来。</p>
<h3 data-nodeid="212755" class="">准备工作</h3>

<p data-nodeid="211185">请确保已经安装好 Docker 并可以正常运行，如果没有安装可以参考 <a href="https://cuiqingcai.com/5438.html" data-nodeid="211273">https://cuiqingcai.com/5438.html</a>。</p>
<h3 data-nodeid="213105" class="">创建 Dockerfile</h3>

<p data-nodeid="211187">首先在项目的根目录下新建一个 requirements.txt 文件，将整个项目依赖的 Python 环境包都列出来，如下所示：</p>
<pre class="lang-java" data-nodeid="214502"><code data-language="java">scrapy 
pymongo 
</code></pre>




<p data-nodeid="211189">如果库需要特定的版本，我们还可以指定版本号，如下所示：</p>
<pre class="lang-java" data-nodeid="214851"><code data-language="java">scrapy&gt;=<span class="hljs-number">1.4</span>.<span class="hljs-number">0</span> 
pymongo&gt;=<span class="hljs-number">3.4</span>.<span class="hljs-number">0</span> 
</code></pre>

<p data-nodeid="211191">在项目根目录下新建一个 Dockerfile 文件，文件不加任何后缀名，修改内容如下所示：</p>
<pre class="lang-java" data-nodeid="215200"><code data-language="java">FROM python:<span class="hljs-number">3.7</span> 
ENV PATH /usr/local/bin:$PATH 
ADD . /code 
WORKDIR /code 
RUN pip3 install -r requirements.txt 
CMD scrapy crawl quotes 
</code></pre>

<p data-nodeid="215549">第一行的 FROM 代表使用的 Docker 基础镜像，在这里我们直接使用 python:3.7 的镜像，在此基础上运行 Scrapy 项目。</p>
<p data-nodeid="215550">第二行 ENV 是环境变量设置，将 /usr/local/bin:$PATH 赋值给 PATH，即增加 /usr/local/bin 这个环境的变量路径。</p>

<p data-nodeid="211194">第三行 ADD 是将本地的代码放置到虚拟容器中。它有两个参数：第一个参数是“.”，代表本地当前路径；第二个参数是 /code，代表虚拟容器中的路径，也就是将本地项目所有内容放置到虚拟容器的 /code 目录下，以便于在虚拟容器中运行代码。</p>
<p data-nodeid="211195">第四行 WORKDIR 是指定工作目录，这里将刚才添加的代码路径设置成工作路径。在这个路径下的目录结构和当前本地目录结构是相同的，所以我们可以直接执行库安装命令、爬虫运行命令等。</p>
<p data-nodeid="211196">第五行 RUN 是执行某些命令来做一些环境准备工作。由于 Docker 虚拟容器内只有 Python 3 环境，而没有所需要的 Python 库，所以我们运行此命令在虚拟容器中安装相应的 Python 库如 Scrapy，这样就可以在虚拟容器中执行 Scrapy 命令了。</p>
<p data-nodeid="211197">第六行 CMD 是容器启动命令。在容器运行时，此命令会被执行。在这里我们直接用 scrapy crawl quotes 来启动爬虫。</p>
<h3 data-nodeid="215901" class="">修改 MongoDB 连接</h3>

<p data-nodeid="211199">接下来我们需要修改 MongoDB 的连接信息。如果我们继续用 localhost 是无法找到 MongoDB 的，因为在 Docker 虚拟容器里 localhost 实际指向容器本身的运行 IP，而容器内部并没有安装 MongoDB，所以爬虫无法连接 MongoDB。</p>
<p data-nodeid="211200">这里的 MongoDB 地址可以有如下两种选择。</p>
<ul data-nodeid="211201">
<li data-nodeid="211202">
<p data-nodeid="211203">如果只想在本机测试，我们可以将地址修改为宿主机的 IP，也就是容器外部的本机 IP，一般是一个局域网 IP，使用 ifconfig 命令即可查看。</p>
</li>
<li data-nodeid="211204">
<p data-nodeid="211205">如果要部署到远程主机运行，一般 MongoDB 都是可公网访问的地址，修改为此地址即可。</p>
</li>
</ul>
<p data-nodeid="211206">但为了保证灵活性，我们可以将这个连接字符串通过环境变量传递进来，比如修改为：</p>
<pre class="lang-java" data-nodeid="216251"><code data-language="java"><span class="hljs-keyword">import</span> os 
​ 
MONGO_URI = os.getenv(<span class="hljs-string">'MONGO_URI'</span>) 
MONGO_DB = os.getenv(<span class="hljs-string">'MONGO_DB'</span>, <span class="hljs-string">'tutorial'</span>) 
</code></pre>

<p data-nodeid="211208">这样项目的配置就完成了。</p>
<h3 data-nodeid="216950" class="">构建镜像</h3>


<p data-nodeid="211210">接下来，我们便可以构建镜像了，执行如下命令：</p>
<pre class="lang-java" data-nodeid="217300"><code data-language="java">docker build -t quotes:latest . 
</code></pre>

<p data-nodeid="211212">这样输出就说明镜像构建成功。这时我们查看一下构建的镜像，如下所示：</p>
<pre class="lang-java" data-nodeid="217649"><code data-language="java">Sending build context to Docker daemon <span class="hljs-number">191.5</span> kB 
Step <span class="hljs-number">1</span>/<span class="hljs-number">6</span> : FROM python:<span class="hljs-number">3.7</span> 
 ---&gt; <span class="hljs-number">968120d</span>8cbe8 
Step <span class="hljs-number">2</span>/<span class="hljs-number">6</span> : ENV PATH /usr/local/bin:$PATH 
 ---&gt; Using cache 
 ---&gt; <span class="hljs-number">387</span>abbba1189 
Step <span class="hljs-number">3</span>/<span class="hljs-number">6</span> : ADD . /code 
 ---&gt; a844ee0db9c6 
Removing intermediate container <span class="hljs-number">4d</span>c41779c573 
Step <span class="hljs-number">4</span>/<span class="hljs-number">6</span> : WORKDIR /code 
 ---&gt; <span class="hljs-number">619</span>b2c064ae9 
Removing intermediate container bcd7cd7f7337 
Step <span class="hljs-number">5</span>/<span class="hljs-number">6</span> : RUN pip3 install -r requirements.txt 
 ---&gt; Running in <span class="hljs-number">9452</span>c83a12c5 
... 
Removing intermediate container <span class="hljs-number">9452</span>c83a12c5 
Step <span class="hljs-number">6</span>/<span class="hljs-number">6</span> : CMD scrapy crawl quotes 
 ---&gt; Running in c092b5557ab8 
 ---&gt; c8101aca6e2a 
Removing intermediate container c092b5557ab8 
Successfully built c8101aca6e2a 
</code></pre>

<p data-nodeid="211214">出现类似输出就证明镜像构建成功了，这时执行，比如我们查看一下构建的镜像：</p>
<pre class="lang-java" data-nodeid="217998"><code data-language="java">docker images 
</code></pre>

<p data-nodeid="211216">返回结果中其中有一行就是：</p>
<pre class="lang-java" data-nodeid="218347"><code data-language="java">quotes  latest  <span class="hljs-number">41</span>c8499ce210 &nbsp;  <span class="hljs-number">2</span> minutes ago &nbsp; <span class="hljs-number">769</span> MB 
</code></pre>

<p data-nodeid="211218">这就是我们新构建的镜像。</p>
<h3 data-nodeid="219046" class="">运行</h3>


<p data-nodeid="211220">我们可以先在本地测试运行，这时候我们需要指定 MongoDB 的连接信息，比如我在宿主机上启动了一个 MongoDB，找到当前宿主机的 IP 为 192.168.3.47，那么这里我就可以指定 MONGO_URI 并启动 Docker 镜像：</p>
<pre class="lang-java" data-nodeid="219396"><code data-language="java">docker run -e MONGO_URI=<span class="hljs-number">192.168</span>.<span class="hljs-number">3.47</span> quotes 
</code></pre>

<p data-nodeid="220094">当然我们还可以指定 MONGO_URI 为远程 MongoDB 连接字符串。</p>
<p data-nodeid="220095">另外我们也可以利用 Docker-Compose 来启动，与此同时顺便也可以使用 Docker 来新建一个 MongoDB。可以在项目目录下新建 docker-compose.yaml 文件，如下所示：</p>

<pre class="lang-java" data-nodeid="219745"><code data-language="java">version: <span class="hljs-string">'3'</span> 
services: 
  crawler: 
 &nbsp;  build: . 
 &nbsp;  image: quotes 
 &nbsp;  depends_on: 
 &nbsp; &nbsp;  - mongo 
 &nbsp;  environment: 
 &nbsp; &nbsp;  MONGO_URI: mongo:<span class="hljs-number">7017</span> 
  mongo: 
 &nbsp;  image: mongo 
 &nbsp;  ports: 
 &nbsp; &nbsp;  - <span class="hljs-number">7017</span>:<span class="hljs-number">27017</span> 
</code></pre>

<p data-nodeid="220630">这里我们使用 Docker-Compose 配置了两个容器，二者需要配合启动。</p>
<p data-nodeid="220631">首先是 crawler 这个容器，其 build 路径是当前路径，image 代表 build 生成的镜像名称，这里取名为 quotes，depends_on 代表容器的启动依赖顺序，这里依赖于 mongo 这个容器，environment 这里就是指定容器运行时的环境变量，这里指定为 <code data-backticks="1" data-nodeid="220636">mongo:7017</code> 。</p>



<p data-nodeid="211225">另外一个容器就是刚才的 crawler 这个容器所依赖的 MongoDB 数据库了，在这里我们直接指定了镜像为 mongo，运行端口配置为 <code data-backticks="1" data-nodeid="211316">7017:27017</code> ，这代表容器内的 MongoDB 运行在 27017 端口上，这个端口会映射到宿主机的 7017 端口上，所以我们在宿主机通过 7017 端口就能连接到这个 MongoDB 数据库。</p>
<p data-nodeid="211226">好，这时候我们运行一下：</p>
<pre class="lang-java" data-nodeid="220986"><code data-language="java">docker-compose up 
</code></pre>

<p data-nodeid="211228">然后 Docker 会构建镜像并运行，运行结果如下：</p>
<pre class="lang-java" data-nodeid="221335"><code data-language="java">Starting scrapytutorial_mongo_1 ... done 
Recreating scrapytutorial_crawler_1 ... done 
Attaching to scrapytutorial_mongo_1, scrapytutorial_crawler_1 
mongo_1 &nbsp;  | {<span class="hljs-string">"t"</span>:{<span class="hljs-string">"$date"</span>:<span class="hljs-string">"2020-08-06T16:18:05.310+00:00"</span>},<span class="hljs-string">"s"</span>:<span class="hljs-string">"I"</span>,  <span class="hljs-string">"c"</span>:<span class="hljs-string">"CONTROL"</span>,  <span class="hljs-string">"id"</span>:<span class="hljs-number">23285</span>, &nbsp; <span class="hljs-string">"ctx"</span>:<span class="hljs-string">"main"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"</span>} 
mongo_1 &nbsp;  | {<span class="hljs-string">"t"</span>:{<span class="hljs-string">"$date"</span>:<span class="hljs-string">"2020-08-06T16:18:05.312+00:00"</span>},<span class="hljs-string">"s"</span>:<span class="hljs-string">"W"</span>,  <span class="hljs-string">"c"</span>:<span class="hljs-string">"ASIO"</span>, &nbsp; &nbsp; <span class="hljs-string">"id"</span>:<span class="hljs-number">22601</span>, &nbsp; <span class="hljs-string">"ctx"</span>:<span class="hljs-string">"main"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"No TransportLayer configured during NetworkInterface startup"</span>} 
mongo_1 &nbsp;  | {<span class="hljs-string">"t"</span>:{<span class="hljs-string">"$date"</span>:<span class="hljs-string">"2020-08-06T16:18:05.312+00:00"</span>},<span class="hljs-string">"s"</span>:<span class="hljs-string">"I"</span>,  <span class="hljs-string">"c"</span>:<span class="hljs-string">"NETWORK"</span>,  <span class="hljs-string">"id"</span>:<span class="hljs-number">4648601</span>, <span class="hljs-string">"ctx"</span>:<span class="hljs-string">"main"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"Implicit TCP FastOpen unavailable. If TCP FastOpen is required, set tcpFastOpenServer, tcpFastOpenClient, and tcpFastOpenQueueSize."</span>} 
... 
crawler_1  | <span class="hljs-number">2020</span>-<span class="hljs-number">08</span>-<span class="hljs-number">06</span> <span class="hljs-number">16</span>:<span class="hljs-number">18</span>:<span class="hljs-number">06</span> [scrapy.utils.log] INFO: Scrapy <span class="hljs-number">2.3</span>.<span class="hljs-number">0</span> started (bot: tutorial) 
crawler_1  | <span class="hljs-number">2020</span>-<span class="hljs-number">08</span>-<span class="hljs-number">06</span> <span class="hljs-number">16</span>:<span class="hljs-number">18</span>:<span class="hljs-number">06</span> [scrapy.utils.log] INFO: Versions: lxml <span class="hljs-number">4.5</span>.<span class="hljs-number">2.0</span>, libxml2 <span class="hljs-number">2.9</span>.<span class="hljs-number">10</span>, cssselect <span class="hljs-number">1.1</span>.<span class="hljs-number">0</span>, parsel <span class="hljs-number">1.6</span>.<span class="hljs-number">0</span>, w3lib <span class="hljs-number">1.22</span>.<span class="hljs-number">0</span>, Twisted <span class="hljs-number">20.3</span>.<span class="hljs-number">0</span>, Python <span class="hljs-number">3.7</span>.<span class="hljs-number">8</span> (<span class="hljs-keyword">default</span>, Jun <span class="hljs-number">30</span> <span class="hljs-number">2020</span>, <span class="hljs-number">18</span>:<span class="hljs-number">27</span>:<span class="hljs-number">23</span>) - [GCC <span class="hljs-number">8.3</span>.<span class="hljs-number">0</span>], pyOpenSSL <span class="hljs-number">19.1</span>.<span class="hljs-number">0</span> (OpenSSL <span class="hljs-number">1.1</span>.<span class="hljs-number">1</span>g  <span class="hljs-number">21</span> Apr <span class="hljs-number">2020</span>), cryptography <span class="hljs-number">3.0</span>, Platform Linux-<span class="hljs-number">4.19</span>.<span class="hljs-number">76</span>-linuxkit-x86_64-with-debian-<span class="hljs-number">10.4</span> 
crawler_1  | <span class="hljs-number">2020</span>-<span class="hljs-number">08</span>-<span class="hljs-number">06</span> <span class="hljs-number">16</span>:<span class="hljs-number">18</span>:<span class="hljs-number">06</span> [scrapy.utils.log] DEBUG: Using reactor: twisted.internet.epollreactor.EPollReactor 
crawler_1  | <span class="hljs-number">2020</span>-<span class="hljs-number">08</span>-<span class="hljs-number">06</span> <span class="hljs-number">16</span>:<span class="hljs-number">18</span>:<span class="hljs-number">06</span> [scrapy.crawler] INFO: Overridden settings: 
crawler_1  | {<span class="hljs-string">'BOT_NAME'</span>: <span class="hljs-string">'tutorial'</span>, 
</code></pre>

<p data-nodeid="222026">这时候就发现爬虫已经正常运行了，同时我们在宿主机上连接 localhost:7017 这个 MongoDB 服务就能看到爬取的结果了：</p>
<p data-nodeid="222027" class=""><img src="https://s0.lgstatic.com/i/image/M00/3E/C2/CgqCHl8tIB6AJUkZAAMRIUmdoXQ641.png" alt="Drawing 0.png" data-nodeid="222031"></p>


<p data-nodeid="211231">这就是用 Docker-Compose 启动的方式，其启动更加便捷，参数可以配置到 Docker-Compose 文件中。</p>
<h3 data-nodeid="222380" class="">推送至 Docker Hub</h3>

<p data-nodeid="211233">构建完成之后，我们可以将镜像 Push 到 Docker 镜像托管平台，如 Docker Hub 或者私有的 Docker Registry 等，这样我们就可以从远程服务器下拉镜像并运行了。</p>
<p data-nodeid="211234">以 Docker Hub 为例，如果项目包含一些私有的连接信息（如数据库），我们最好将 Repository 设为私有或者直接放到私有的 Docker Registry 中。</p>
<p data-nodeid="211235">首先在 <a href="https://hub.docker.com" data-nodeid="211332">https://hub.docker.com</a>注册一个账号，新建一个 Repository，名为 quotes。比如，我的用户名为 germey，新建的 Repository 名为 quotes，那么此 Repository 的地址就可以用 germey/quotes 来表示，当然你也可以自行修改。</p>
<p data-nodeid="211236">为新建的镜像打一个标签，命令如下所示：</p>
<pre class="lang-java" data-nodeid="222730"><code data-language="java">docker tag quotes:latest germey/quotes:latest 
</code></pre>

<p data-nodeid="211238">推送镜像到 Docker Hub 即可，命令如下所示：</p>
<pre class="lang-java" data-nodeid="223079"><code data-language="java">docker push germey/quotes 
</code></pre>

<p data-nodeid="223770">Docker Hub 中便会出现新推送的 Docker 镜像了，如图所示。</p>
<p data-nodeid="223771" class=""><img src="https://s0.lgstatic.com/i/image/M00/3E/C2/CgqCHl8tIC6AZRf7AAHlmVEMa3U133.png" alt="Drawing 1.png" data-nodeid="223775"></p>


<p data-nodeid="211241">如果我们想在其他的主机上运行这个镜像，在主机上装好 Docker 后，可以直接执行如下命令：</p>
<pre class="lang-java" data-nodeid="224124"><code data-language="java">docker run germey/quotes 
</code></pre>

<p data-nodeid="224822">这样就会自动下载镜像，然后启动容器运行，不需要配置 Python 环境，不需要关心版本冲突问题。</p>
<p data-nodeid="224823">当然我们也可以使用 Docker-Compose 来构建镜像和推送镜像，这里我们只需要修改 docker-compose.yaml 文件即可：</p>

<pre class="lang-java" data-nodeid="224473"><code data-language="java">version: <span class="hljs-string">'3'</span> 
services: 
  crawler: 
 &nbsp;  build: . 
 &nbsp;  image: germey/quotes 
 &nbsp;  depends_on: 
 &nbsp; &nbsp;  - mongo 
 &nbsp;  environment: 
 &nbsp; &nbsp;  MONGO_URI: mongo:<span class="hljs-number">7017</span> 
  mongo: 
 &nbsp;  image: mongo 
 &nbsp;  ports: 
 &nbsp; &nbsp;  - <span class="hljs-number">7017</span>:<span class="hljs-number">27017</span> 
</code></pre>

<p data-nodeid="211245">可以看到，这里我们就将 crawler 的 image 内容修改为了 <code data-backticks="1" data-nodeid="211346">germey/quotes</code> ，接下来执行：</p>
<pre class="lang-java" data-nodeid="225174"><code data-language="java">docker-compose build 
docker-compose push 
</code></pre>

<p data-nodeid="211247">就可以把镜像推送到 Docker Hub 了。</p>
<h3 data-nodeid="225523" class="">结语</h3>

<p data-nodeid="211249">本课时，我们讲解了将 Scrapy 项目制作成 Docker 镜像并部署到远程服务器运行的过程。使用此种方式，我们在本节课开始时所列出的问题都可以迎刃而解了。</p>

---

### 精选评论


