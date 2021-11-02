<p data-nodeid="1613" class="">前几讲我们都使用了一种非常简单暴力的方式（node app.js）启动 Node.js 服务器，而在线上我们要考虑使用多核 CPU，充分利用服务器资源，这里就用到多进程解决方案，所以本讲介绍 PM2 的原理以及如何应用一个 cluster 模式启动 Node.js 服务。</p>
<h3 data-nodeid="1614">单线程问题</h3>
<p data-nodeid="1615">在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=694#/detail/pc?id=6783" data-nodeid="1750">《01 | 事件循环：高性能到底是如何做到的？》</a>中我们分析了 Node.js 主线程是单线程的，如果我们使用 node app.js 方式运行，就启动了一个进程，只能在<strong data-nodeid="1764">一个 CPU 中进行运算</strong>，无法应用服务器的多核 CPU，因此我们需要寻求一些解决方案。你能想到的解决方案肯定是<strong data-nodeid="1765">多进程分发策略</strong>，即主进程接收所有请求，然后通过一定的<strong data-nodeid="1766">负载均衡策略</strong>分发到不同的 Node.js 子进程中。如图 1 的方案所示：</p>
<p data-nodeid="1616"><img src="https://s0.lgstatic.com/i/image6/M01/1D/E0/CioPOWBQKV2ABtnsAAAuF7ZUkEQ818.png" alt="Drawing 1.png" data-nodeid="1769"></p>
<p data-nodeid="1617">这一方案有 2 个不同的实现：</p>
<ul data-nodeid="1618">
<li data-nodeid="1619">
<p data-nodeid="1620">主进程监听一个端口，子进程不监听端口，通过主进程分发请求到子进程；</p>
</li>
<li data-nodeid="1621">
<p data-nodeid="1622">主进程和子进程分别监听不同端口，通过主进程分发请求到子进程。</p>
</li>
</ul>
<p data-nodeid="1623">在 Node.js 中的 cluster 模式使用的是第一个实现。</p>
<h3 data-nodeid="1624">cluster 模式</h3>
<p data-nodeid="1625">cluster 模式其实就是我们上面图 1 所介绍的模式，<strong data-nodeid="1784">一个主进程</strong>和<strong data-nodeid="1785">多个子进程</strong>，从而形成一个集群的概念。我们先来看看 cluster 模式的应用例子。</p>
<h4 data-nodeid="1626">应用</h4>
<p data-nodeid="1627">我们先实现一个简单的 app.js，代码如下：</p>
<pre class="lang-javascript" data-nodeid="1628"><code data-language="javascript"><span class="hljs-keyword">const</span> http = <span class="hljs-built_in">require</span>(<span class="hljs-string">'http'</span>);
<span class="hljs-comment">/**
 * 
 * 创建 http 服务，简单返回
 */</span>
<span class="hljs-keyword">const</span> server = http.createServer(<span class="hljs-function">(<span class="hljs-params">req, res</span>) =&gt;</span> {
    res.write(<span class="hljs-string">`hello world, start with cluster <span class="hljs-subst">${process.pid}</span>`</span>);
    res.end();
});
<span class="hljs-comment">/**
 * 
 * 启动服务
 */</span>
server.listen(<span class="hljs-number">3000</span>, () =&gt; {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'server start http://127.0.0.1:3000'</span>);
});
<span class="hljs-built_in">console</span>.log(<span class="hljs-string">`Worker <span class="hljs-subst">${process.pid}</span> started`</span>);
</code></pre>
<p data-nodeid="1629">这是最简单的一个 Node.js 服务，接下来我们应用 cluster 模式来包装这个服务，代码如下：</p>
<pre class="lang-javascript" data-nodeid="1630"><code data-language="javascript"><span class="hljs-keyword">const</span> cluster = <span class="hljs-built_in">require</span>(<span class="hljs-string">'cluster'</span>);
<span class="hljs-keyword">const</span> instances = <span class="hljs-number">2</span>; <span class="hljs-comment">// 启动进程数量</span>
<span class="hljs-keyword">if</span> (cluster.isMaster) {
    <span class="hljs-keyword">for</span>(<span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>;i&lt;instances;i++) { <span class="hljs-comment">// 使用 cluster.fork 创建子进程</span>
        cluster.fork();
    }
} <span class="hljs-keyword">else</span> {
    <span class="hljs-built_in">require</span>(<span class="hljs-string">'./app.js'</span>);
}
</code></pre>
<p data-nodeid="1631">首先判断是否为主进程：</p>
<ul data-nodeid="1632">
<li data-nodeid="1633">
<p data-nodeid="1634">如果是则使用 cluster.fork 创建子进程；</p>
</li>
<li data-nodeid="1635">
<p data-nodeid="1636">如果不是则为子进程 require 具体的 app.js。</p>
</li>
</ul>
<p data-nodeid="1637">然后运行下面命令启动服务。</p>
<pre class="lang-java" data-nodeid="1638"><code data-language="java">$ node cluster.js
</code></pre>
<p data-nodeid="1639">启动成功后，再打开另外一个命令行窗口，多次运行以下命令：</p>
<pre class="lang-java" data-nodeid="1640"><code data-language="java">curl <span class="hljs-string">"http://127.0.0.1:3000/"</span>
</code></pre>
<p data-nodeid="1641">你可以看到如下输出：</p>
<pre class="lang-java" data-nodeid="1642"><code data-language="java">hello world, start with cluster <span class="hljs-number">4543</span>
hello world, start with cluster <span class="hljs-number">4542</span>
hello world, start with cluster <span class="hljs-number">4543</span>
hello world, start with cluster <span class="hljs-number">4542</span>
</code></pre>
<p data-nodeid="1643">后面的进程 ID 是比较有规律的随机数，有时候输出 4543，有时候输出 4542，4543 和 4542 就是我们 <strong data-nodeid="1800">fork 出来的两个子进程</strong>，接下来我们看下为什么是这样的。</p>
<h4 data-nodeid="1644">原理</h4>
<p data-nodeid="1645">首先我们需要搞清楚两个问题：</p>
<ul data-nodeid="1646">
<li data-nodeid="1647">
<p data-nodeid="1648">Node.js 的 cluster 是如何做到多个进程监听一个端口的；</p>
</li>
<li data-nodeid="1649">
<p data-nodeid="1650">Node.js 是如何进行负载均衡请求分发的。</p>
</li>
</ul>
<p data-nodeid="1651"><strong data-nodeid="1808">多进程端口问题</strong></p>
<p data-nodeid="1652">在 cluster 模式中存在 master 和 worker 的概念，<strong data-nodeid="1818">master 就是主进程</strong>，<strong data-nodeid="1819">worker 则是子进程</strong>，因此这里我们需要看下 master 进程和 worker 进程的创建方式。如下代码所示：</p>
<pre class="lang-javascript" data-nodeid="1653"><code data-language="javascript"><span class="hljs-keyword">const</span> cluster = <span class="hljs-built_in">require</span>(<span class="hljs-string">'cluster'</span>);
<span class="hljs-keyword">const</span> instances = <span class="hljs-number">2</span>; <span class="hljs-comment">// 启动进程数量</span>
<span class="hljs-keyword">if</span> (cluster.isMaster) {
    <span class="hljs-keyword">for</span>(<span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>;i&lt;instances;i++) { <span class="hljs-comment">// 使用 cluster.fork 创建子进程</span>
        cluster.fork();
    }
} <span class="hljs-keyword">else</span> {
    <span class="hljs-built_in">require</span>(<span class="hljs-string">'./app.js'</span>);
}
</code></pre>
<p data-nodeid="1654">这段代码中，第一次 require 的 cluster 对象就默认是一个 master，这里的判断逻辑在<a href="https://github.com/nodejs/node/blob/master/lib/cluster.js" data-nodeid="1823">源码</a>中，如下代码所示：</p>
<pre class="lang-javascript" data-nodeid="1655"><code data-language="javascript"><span class="hljs-meta">'use strict'</span>;
	
	<span class="hljs-keyword">const</span> childOrPrimary = <span class="hljs-string">'NODE_UNIQUE_ID'</span> <span class="hljs-keyword">in</span> process.env ? <span class="hljs-string">'child'</span> : <span class="hljs-string">'primary'</span>;
	<span class="hljs-built_in">module</span>.exports = <span class="hljs-built_in">require</span>(<span class="hljs-string">`internal/cluster/<span class="hljs-subst">${childOrPrimary}</span>`</span>);
</code></pre>
<p data-nodeid="1656">通过<strong data-nodeid="1830">进程环境变量设置</strong>来判断：</p>
<ul data-nodeid="1657">
<li data-nodeid="1658">
<p data-nodeid="1659">如果没有设置则为 master 进程；</p>
</li>
<li data-nodeid="1660">
<p data-nodeid="1661">如果有设置则为子进程。</p>
</li>
</ul>
<p data-nodeid="1662">因此第一次调用 cluster 模块是 master 进程，而后都是子进程。</p>
<p data-nodeid="1663">主进程和子进程 require 文件不同：</p>
<ul data-nodeid="1664">
<li data-nodeid="1665">
<p data-nodeid="1666">前者是 internal/cluster/primary；</p>
</li>
<li data-nodeid="1667">
<p data-nodeid="1668">后者是 internal/cluster/child。</p>
</li>
</ul>
<p data-nodeid="1669">我们先来看下 master 进程的创建过程，这部分<a href="https://github.com/nodejs/node/blob/7397c7e4a303b1ebad84892872717c0092852921/lib/internal/cluster/primary.js#L60" data-nodeid="1840">代码在这里</a>。</p>
<p data-nodeid="1670">可以看到 cluster.fork，一开始就会调用 setupPrimary 方法，创建主进程，由于该方法是通过 cluster.fork 调用，因此会调用多次，但是该模块有个全局变量 initialized 用来区分是否为首次，如果是首次则创建，否则则跳过，如下代码：</p>
<pre class="lang-javascript" data-nodeid="1671"><code data-language="javascript">  <span class="hljs-keyword">if</span> (initialized === <span class="hljs-literal">true</span>)
	    <span class="hljs-keyword">return</span> process.nextTick(setupSettingsNT, settings);
	
	  initialized = <span class="hljs-literal">true</span>;
</code></pre>
<p data-nodeid="1672">接下来继续看 cluster.fork 方法，源码如下：</p>
<pre class="lang-javascript" data-nodeid="1673"><code data-language="javascript">cluster.fork = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">env</span>) </span>{
	  cluster.setupPrimary();
	  <span class="hljs-keyword">const</span> id = ++ids;
	  <span class="hljs-keyword">const</span> workerProcess = createWorkerProcess(id, env);
	  <span class="hljs-keyword">const</span> worker = <span class="hljs-keyword">new</span> Worker({
	    <span class="hljs-attr">id</span>: id,
	    <span class="hljs-attr">process</span>: workerProcess
	  });
	
	  worker.on(<span class="hljs-string">'message'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">message, handle</span>) </span>{
	    cluster.emit(<span class="hljs-string">'message'</span>, <span class="hljs-keyword">this</span>, message, handle);
	  });
</code></pre>
<p data-nodeid="1674">在上面代码中第 2 行就是<strong data-nodeid="1855">创建主进程</strong>，第 4 行就是<strong data-nodeid="1856">创建 worker 子进程</strong>，在这个 createWorkerProcess 方法中，最终是使用 child_process 来创建子进程的。在初始化代码中，我们调用了两次 cluster.fork 方法，因此会创建 2 个子进程，在创建后又会调用我们项目根目录下的 cluster.js 启动一个新实例，这时候由于 cluster.isMaster 是 false，因此会 require 到 internal/cluster/child 这个方法。</p>
<p data-nodeid="1675">由于是 worker 进程，因此代码会 require ('./app.js') 模块，在该模块中会监听具体的端口，代码如下：</p>
<pre class="lang-javascript" data-nodeid="1676"><code data-language="javascript"><span class="hljs-comment">/**
 * 
 * 启动服务
 */</span>
server.listen(<span class="hljs-number">3000</span>, () =&gt; {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'server start http://127.0.0.1:3000'</span>);
});
<span class="hljs-built_in">console</span>.log(<span class="hljs-string">`Worker <span class="hljs-subst">${process.pid}</span> started`</span>);
</code></pre>
<p data-nodeid="1677">这里的 server.listen 方法很重要，这部分<a href="https://github.com/nodejs/node/blob/15164cebcebfcad9822d3f065234a8c1511776a4/lib/net.js" data-nodeid="1865">源代码在这里</a>，其中的 server.listen 会调用该模块中的 listenInCluster 方法，该方法中有一个关键信息，如下代码所示：</p>
<pre class="lang-javascript" data-nodeid="1678"><code data-language="javascript"><span class="hljs-keyword">if</span> (cluster.isPrimary || exclusive) {
	    <span class="hljs-comment">// Will create a new handle</span>
	    <span class="hljs-comment">// _listen2 sets up the listened handle, it is still named like this</span>
	    <span class="hljs-comment">// to avoid breaking code that wraps this method</span>
	    server._listen2(address, port, addressType, backlog, fd, flags);
	    <span class="hljs-keyword">return</span>;
	  }
	
	  <span class="hljs-keyword">const</span> serverQuery = {
	    <span class="hljs-attr">address</span>: address,
	    <span class="hljs-attr">port</span>: port,
	    <span class="hljs-attr">addressType</span>: addressType,
	    <span class="hljs-attr">fd</span>: fd,
	    flags,
	  };
	
	  <span class="hljs-comment">// Get the primary's server handle, and listen on it</span>
	  cluster._getServer(server, serverQuery, listenOnPrimaryHandle);
</code></pre>
<p data-nodeid="1679">上面代码中的第 6 行，判断为<strong data-nodeid="1880">主进程</strong>，就是<strong data-nodeid="1881">真实的监听端口启动服务</strong>，而如果非主进程则调用 cluster._getServer 方法，也就是 internal/cluster/child 中的 cluster._getServer 方法。</p>
<p data-nodeid="1680">接下来我们看下这部分代码：</p>
<pre class="lang-javascript" data-nodeid="1681"><code data-language="javascript">obj.once(<span class="hljs-string">'listening'</span>, () =&gt; {
	    cluster.worker.state = <span class="hljs-string">'listening'</span>;
	    <span class="hljs-keyword">const</span> address = obj.address();
	    message.act = <span class="hljs-string">'listening'</span>;
	    message.port = (address &amp;&amp; address.port) || options.port;
	    send(message);
	  });
</code></pre>
<p data-nodeid="1682">这一代码通过 send 方法，如果监听到 listening 发送一个消息给到主进程，主进程也有一个同样的 listening 事件，监听到该事件后将子进程通过 EventEmitter 绑定在主进程上，这样就完成了主子进程之间的<strong data-nodeid="1892">关联绑定</strong>，并且只监听了一个端口。而主子进程之间的通信方式，就是我们常听到的 <strong data-nodeid="1893">IPC 通信方式</strong>。</p>
<p data-nodeid="1683"><strong data-nodeid="1897">负载均衡原理</strong></p>
<p data-nodeid="1684">既然 Node.js cluster 模块使用的是主子进程方式，那么它是如何进行负载均衡处理的呢，这里就会涉及 Node.js cluster 模块中的两个模块。</p>
<ul data-nodeid="1685">
<li data-nodeid="1686">
<p data-nodeid="1687"><a href="https://github.com/nodejs/node/blob/7397c7e4a303b1ebad84892872717c0092852921/lib/internal/cluster/round_robin_handle.js" data-nodeid="1905">round_robin_handle.js</a>（非 Windows 平台应用模式），这是一个<strong data-nodeid="1911">轮询处理模式</strong>，也就是轮询调度分发给空闲的子进程，处理完成后回到 worker 空闲池子中，这里要注意的就是如果绑定过就会复用该子进程，如果没有则会重新判断，这里可以通过上面的 app.js 代码来测试，用浏览器去访问，你会发现每次调用的子进程 ID 都会不变。</p>
</li>
<li data-nodeid="1688">
<p data-nodeid="1689"><a href="https://github.com/nodejs/node/blob/7397c7e4a303b1ebad84892872717c0092852921/lib/internal/cluster/shared_handle.js" data-nodeid="1916">shared_handle.js</a>（ Windows 平台应用模式），通过将文件描述符、端口等信息传递给子进程，子进程通过信息创建相应的 SocketHandle / ServerHandle，然后进行相应的端口绑定和监听、处理请求。</p>
</li>
</ul>
<p data-nodeid="1690">以上就是 cluster 的原理，总结一下就是 cluster 模块应用 child_process 来创建子进程，子进程通过复写掉 cluster._getServer 方法，从而在 server.listen 来保证只有主进程监听端口，主子进程通过 IPC 进行通信，其次主进程根据平台或者协议不同，应用两种不同模块（round_robin_handle.js 和 shared_handle.js）进行请求分发给子进程处理。接下来我们看一下 cluster 的成熟的应用工具 PM2 的应用和原理。</p>
<h3 data-nodeid="1691">PM2 原理</h3>
<p data-nodeid="1692">PM2 是<strong data-nodeid="1935">守护进程管理器</strong>，可以帮助你管理和保持应用程序在线。PM2 入门非常简单，它是一个简单直观的 CLI 工具，可以通过 NPM 安装，接下来我们看下一些简单的用法。</p>
<h4 data-nodeid="1693">应用</h4>
<p data-nodeid="1694">你可以使用如下命令进行 NPM 或者 Yarn 的安装：</p>
<pre class="lang-java" data-nodeid="1695"><code data-language="java">$ npm install pm2@latest -g
# or
$ yarn global add pm2
</code></pre>
<p data-nodeid="1696">安装成功后，可以使用如下命令查看是否安装成功以及当前的版本：</p>
<pre class="lang-java" data-nodeid="1697"><code data-language="java">$&nbsp;pm2 --version
</code></pre>
<p data-nodeid="1698">接下来我们使用 PM2 启动一个简单的 Node.js 项目，进入本讲代码的项目根目录，然后运行下面命令：</p>
<pre class="lang-java" data-nodeid="1699"><code data-language="java">$&nbsp;pm2 start app.js
</code></pre>
<p data-nodeid="1700">运行后，再执行如下命令：</p>
<pre class="lang-java" data-nodeid="1701"><code data-language="java">$&nbsp;pm2&nbsp;list
</code></pre>
<p data-nodeid="1702">可以看到如图 2 所示的结果，代表运行成功了。</p>
<p data-nodeid="1703"><img src="https://s0.lgstatic.com/i/image6/M01/1D/E3/Cgp9HWBQKZeAM-MIAAB0_RHaw1E022.png" alt="Drawing 2.png" data-nodeid="1944"></p>
<div data-nodeid="1704"><p style="text-align:center">图 2 pm2 list 运行结果</p></div>
<p data-nodeid="1705">PM2 启动时可以带一些配置化参数，具体参数列表你可以参考<a href="https://pm2.keymetrics.io/docs/usage/pm2-doc-single-page/" data-nodeid="1948">官方文档</a>。在开发中我总结出了一套最佳的实践，如以下配置所示：</p>
<pre class="lang-javascript" data-nodeid="1706"><code data-language="javascript"><span class="hljs-built_in">module</span>.exports = {
    <span class="hljs-attr">apps</span> : [{
      <span class="hljs-attr">name</span>: <span class="hljs-string">"nodejs-column"</span>, <span class="hljs-comment">// 启动进程名</span>
      <span class="hljs-attr">script</span>: <span class="hljs-string">"./app.js"</span>, <span class="hljs-comment">// 启动文件</span>
      <span class="hljs-attr">instances</span>: <span class="hljs-number">2</span>, <span class="hljs-comment">// 启动进程数</span>
      <span class="hljs-attr">exec_mode</span>: <span class="hljs-string">'cluster'</span>, <span class="hljs-comment">// 多进程多实例</span>
      <span class="hljs-attr">env_development</span>: {
        <span class="hljs-attr">NODE_ENV</span>: <span class="hljs-string">"development"</span>,
        <span class="hljs-attr">watch</span>: <span class="hljs-literal">true</span>, <span class="hljs-comment">// 开发环境使用 true，其他必须设置为 false</span>
      },
      <span class="hljs-attr">env_testing</span>: {
        <span class="hljs-attr">NODE_ENV</span>: <span class="hljs-string">"testing"</span>,
        <span class="hljs-attr">watch</span>: <span class="hljs-literal">false</span>, <span class="hljs-comment">// 开发环境使用 true，其他必须设置为 false</span>
      },
      <span class="hljs-attr">env_production</span>: {
        <span class="hljs-attr">NODE_ENV</span>: <span class="hljs-string">"production"</span>,
        <span class="hljs-attr">watch</span>: <span class="hljs-literal">false</span>, <span class="hljs-comment">// 开发环境使用 true，其他必须设置为 false</span>
      },
      <span class="hljs-attr">log_date_format</span>: <span class="hljs-string">'YYYY-MM-DD HH:mm Z'</span>,
      <span class="hljs-attr">error_file</span>: <span class="hljs-string">'~/data/err.log'</span>, <span class="hljs-comment">// 错误日志文件，必须设置在项目外的目录，这里为了测试</span>
      <span class="hljs-attr">out_file</span>: <span class="hljs-string">'~/data/info.log'</span>, <span class="hljs-comment">//  流水日志，包括 console.log 日志，必须设置在项目外的目录，这里为了测试</span>
      <span class="hljs-attr">max_restarts</span>: <span class="hljs-number">10</span>,
    }]
  }
</code></pre>
<p data-nodeid="3650" class="te-preview-highlight">在上面的配置中要特别注意 <strong data-nodeid="3664">error_file</strong> 和 <strong data-nodeid="3665">out_file</strong>，这里的日志目录在项目初始化时要创建好，如果不提前创建好会导致线上运行失败，特别是无权限创建目录时。其次如果存在环境差异的配置时，可以放置在不同的环境下，最终可以使用下面三种方式来启动项目，分别对应不同环境。</p>



<pre class="lang-shell" data-nodeid="1708"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> pm2 start pm2.config.js --env development</span>
<span class="hljs-meta">$</span><span class="bash"> pm2 start pm2.config.js --env testing</span>
<span class="hljs-meta">$</span><span class="bash"> pm2 start pm2.config.js --env production</span>
</code></pre>
<h4 data-nodeid="1709">原理</h4>
<p data-nodeid="1710">接下来我们来看下是如何实现的，由于整个项目是比较复杂庞大的，这里我们主要关注<strong data-nodeid="1971">进程创建管理的原理</strong>。</p>
<p data-nodeid="1711">首先我们来看下进程创建的方式，整体的流程如图 3 所示。</p>
<p data-nodeid="1712"><img src="https://s0.lgstatic.com/i/image6/M01/1D/E0/CioPOWBQKaWAHrR1AAKhg2CW1Z0319.png" alt="Drawing 3.png" data-nodeid="1975"></p>
<div data-nodeid="1713"><p style="text-align:center">图 3 PM2 源码多进程创建方式</p></div>
<p data-nodeid="1714">这一方式涉及五个模块文件。</p>
<ul data-nodeid="1715">
<li data-nodeid="1716">
<p data-nodeid="1717">CLI（lib/binaries/CLI.js）处理命令行输入，如我们运行的命令：</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1718"><code data-language="java">pm2 start pm2.config.js --env development
</code></pre>
<ul data-nodeid="1719">
<li data-nodeid="1720">
<p data-nodeid="1721">API（lib/API.js）对外暴露的各种命令行调用方法，比如上面的 start 命令对应的 API-&gt;start 方法。</p>
</li>
<li data-nodeid="1722">
<p data-nodeid="1723">Client （lib/Client.js）可以理解为命令行接收端，负责创建守护进程 Daemon，并与 Daemon（lib/Daemon.js）保持 RPC 连接。</p>
</li>
<li data-nodeid="1724">
<p data-nodeid="1725">God （lib/God.js）主要负责进程的创建和管理，主要是通过 Daemon 调用，Client 所有调用都是通过 RPC 调用 Daemon，然后 Daemon 调用 God 中的方法。</p>
</li>
<li data-nodeid="1726">
<p data-nodeid="1727">最终在 God 中调用 ClusterMode（lib/God/ClusterMode.js）模块，在 ClusterMode 中调用 Node.js 的 cluster.fork 创建子进程。</p>
</li>
</ul>
<p data-nodeid="1728">图 3 中首先通过命令行解析调用 API，API 中的方法基本上是与 CLI 中的命令行一一对应的，API 中的 start 方法会根据传入参数判断是否是调用的方法，一般情况下使用的都是一个 JSON 配置文件，因此调用 API 中的私有方法 _startJson。</p>
<p data-nodeid="1729">接下来就开始在 Client 模块中流转了，在 _startJson 中会调用 executeRemote 方法，该方法会先判断 PM2 的守护进程 Daemon 是否启动，如果没有启动会先调用 Daemon 模块中的方法启动守护进程 RPC 服务，启动成功后再通知 Client 并建立 RPC 通信连接。</p>
<p data-nodeid="1730">成功建立连接后，Client 会发送启动 Node.js 子进程的命令 prepare，该命令传递 Daemon，Daemon 中有一份对应的命令的执行方法，该命令最终会调用 God 中的 prepare 方法。</p>
<p data-nodeid="1731">在 God 中最终会调用 God 文件夹下的 ClusterMode 模块，应用 Node.js 的 cluster.fork 创建子进程，这样就完成了整个启动过程。</p>
<p data-nodeid="1732">综上所述，PM2 通过命令行，使用 RPC 建立 Client 与 Daemon 进程之间的通信，通过 RPC 通信方式，调用 God，从而应用 Node.js 的 cluster.fork 创建子进程的。以上是启动的流程，对于其他命令指令，比如 stop、restart 等，也是一样的通信流转过程，你参照上面的流程分析就可以了，如果遇到任何问题，都可以在留言区与我交流。</p>
<blockquote data-nodeid="1733">
<p data-nodeid="1734">以上的分析你需要参考<a href="https://github.com/Unitech/pm2/tree/64f8ea0f2c31c7d70a415eccc6222547b3664e65" data-nodeid="1994">PM2 的 GitHub 源码</a>。</p>
</blockquote>
<h3 data-nodeid="1735">总结</h3>
<p data-nodeid="1736">本讲主要介绍了 Node.js 中的 cluster 模块，并深入介绍了其核心原理，其次介绍了目前比较常用的多进程管理工具 PM2 的应用和原理。学完本讲后，需要掌握 Node.js cluster 原理，并且掌握 PM2 的实现原理。</p>
<p data-nodeid="1737">接下来我们将开始讲解一些关于 Node.js 性能相关的知识，为后续的高性能服务做一定的准备，其次也在为后续性能优化打下一定的技术基础。</p>
<p data-nodeid="1738">下一讲会讲解，目前我们在使用的 Node.js cluster 模式存在的性能问题。</p>
<hr data-nodeid="1739">
<p data-nodeid="1740"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="2004"><img src="https://s0.lgstatic.com/i/image6/M00/12/FA/CioPOWBBrAKAAod-AASyC72ZqWw233.png" alt="Drawing 2.png" data-nodeid="2003"></a></p>
<p data-nodeid="1741"><strong data-nodeid="2008">《大前端高薪训练营》</strong></p>
<p data-nodeid="1742" class="">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="2012">点击链接</a>，快来领取！</p>

---

### 精选评论

##### **6400：
> 老师，我想问下进程，线程，实例和cpu核数的关系。1.实例值的就是一台PC,一台服务器吗?2.进程数和CPU核数有什么联系，一个CPU核可以有几个进程，几个线程呢？3.假如我的服务器有8个实例，每个实例2核Cpu,如何配置pm2可以发挥最大性能呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 这个应该叫做一个服务会更好一些，实例在我们里面所说的是一个子进程；
2. 一般 CPU 核数和子进程个数是没有关联的，但是在应用时最好单个服务起的 Node.js 进程不能等于或超过 CPU 核数，如果大于等于时，当Node.js进程跑满时，就会导致 CPU 超负荷，从而机器瘫痪的现象。
3. 8 个服务，每个服务2个进程（一个进程占用一个 CPU 核数），这其实要看你机器的配置，以及每个服务的压力情况。举个例子，假设你服务器只有 4 核，并且其中有 2 个服务请求压力并发较大，你如果配置的这2个服务正好是2个进程，那么很有可能导致整个服务器资源跑满的情况。其实 PM2 背后还是 cluster 模式，而 cluster 模式下你需要根据当前机器的配置，以及你服务的压力来判断要启用多少个子进程，但是核心是不要超出当前 CPU 核数。

##### **喝王老吉：
> 想问一下老师，对于大文件的读写，比如几个G的日志，要分析，会内存溢出，应该怎么办？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 最简单的就是文件拆分，超过一定文件大小时自动拆分为多个文件，其次就是使用文件流。

##### **0971：
> 不是很理解，一个主进程+两个子进程，那就是3个进程，但pm2 list时我们只会看到两个子进程，而且不少文章都是根据cpu个数来创建子进程，那加上主进程，不就比cpu还多了吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; pm2是只给你看了子进程，并不是所有的进程，pm2 自身就有一个独立的进程。为了更清晰，你可以把我们源码中的 cluster.js 在运行一次，使用 node cluster.js ，然后你使用 ps -ef | grep cluster.js | grep -v grep 。你可以看下有几个进程信息，其次你可以看下每个进程的PID 以及它父PID。

##### **森：
> 源代码中的 cluster.js 为什么执行了三次？if里面执行了一次，else里面执行了两次

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的哈。
https://github.com/nodejs/node/blob/master/lib/internal/cluster/primary.js
这里是调用 cluster.fork 的地方，这部分会调用createWorkerProcess创建子进程，这个方法中会调用 fork 方法，而 fork 在文件头部有申明const { fork } = require('child_process');因此最终会将启动参数命令行传递给 child_process 的 fork 方法来启动一个新的进程，再细致点，你可以看到它会把启动相关的参数都传递给 fork 这个函数。具体可以看下这个方法的官方介绍 https://nodejs.org/api/child_process.html#child_process_child_process_fork_modulepath_args_options 。
----
因为问的同学比较多，我在补充下
1. cluster.js 首次运行的时候，isMaster 肯定是 true 这时候会创建一个父进程
2. 在我们代码中调用了两次的 cluster.fork() 方法
3. cluster.fork() 源码中会调用 child_process 的 fork 方法来启动新的进程，在调用 cluster.fork() 时会将进程运行参数全部传递给 child_process fork() 方法，通过这个方法启动了一个新进程；
4. child_process.fork() 的时候会重新运行 node 的启动命令，这时候就会重新运行 cluster.js 这个文件，而这时候 isMaster 是 false 所以会去 requre('app.js') 了。这里还有一个点，就是在 fork 子进程的时候，如果父进程还没有创建，会创建父进程。
5. 在 app.js 中虽然是启动了监听端口，由于监听端口的方法被重写了，因此只是向主进程发送了一个消息，告诉父进程可以向我发送消息了，因此可以一个端口多个进程来服务。

##### **南：
> 引用 ‘cluster.js 为啥执行三次’这个问题，我通过专栏的例子去测试，创建了cluster之后启动，我在cluster.js里面打印了 “cluster.isMaster”的值，在通过 node 命令启动后，打印了三次，一次 true，两次 false，是每一次进程的启动都会触发cluster.js 这个文件的执行吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的哈。
https://github.com/nodejs/node/blob/master/lib/internal/cluster/primary.js
这里是调用 cluster.fork 的地方，这部分会调用createWorkerProcess创建子进程，这个方法中会调用 fork 方法，而 fork 在文件头部有申明const { fork } = require('child_process');因此最终会将启动参数命令行传递给 child_process 的 fork 方法来启动一个新的进程，再细致点，你可以看到它会把启动相关的参数都传递给 fork 这个函数。具体可以看下这个方法的官方介绍 https://nodejs.org/api/child_process.html#child_process_child_process_fork_modulepath_args_options 。
----
因为问的同学比较多，我在补充下
1. cluster.js 首次运行的时候，isMaster 肯定是 true 这时候会创建一个父进程
2. 在我们代码中调用了两次的 cluster.fork() 方法
3. cluster.fork() 源码中会调用 child_process 的 fork 方法来启动新的进程，在调用 cluster.fork() 时会将进程运行参数全部传递给 child_process fork() 方法，通过这个方法启动了一个新进程；
4. child_process.fork() 的时候会重新运行 node 的启动命令，这时候就会重新运行 cluster.js 这个文件，而这时候 isMaster 是 false 所以会去 requre('app.js') 了。这里还有一个点，就是在 fork 子进程的时候，如果父进程还没有创建，会创建父进程。
5. 在 app.js 中虽然是启动了监听端口，由于监听端口的方法被重写了，因此只是向主进程发送了一个消息，告诉父进程可以向我发送消息了，因此可以一个端口多个进程来服务。

##### **杰：
> 不是很明白，直接运行cluster.js的时候，isMaster的值是true还是false。如果是true，那怎么会进入require app.js去运行程序在3000端口呢？如果是false，那什么时候去创建子进程呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是这样的 
1. cluster.js 首次运行的时候，isMaster 肯定是 true 这时候会创建一个父进程
2. 在我们代码中调用了两次的 cluster.fork() 方法
3. cluster.fork() 源码中会调用 child_process 的 fork 方法来启动新的进程，在调用 cluster.fork() 时会将进程运行参数全部传递给 child_process fork() 方法，通过这个方法启动了一个新进程；
4. child_process.fork() 的时候会重新运行 node 的启动命令，这时候就会重新运行 cluster.js 这个文件，而这时候 isMaster 是 false 所以会去 requre('app.js') 了。这里还有一个点，就是在 fork 子进程的时候，如果父进程还没有创建，会创建父进程。
5. 在 app.js 中虽然是启动了监听端口，由于监听端口的方法被重写了，因此只是向主进程发送了一个消息，告诉父进程可以向我发送消息了，因此可以一个端口多个进程来服务。

##### lastbee：
> cluster.js如果console的话，会出现三次结果，isMaster一次为true，另外两次为false。是调用子进程的fork引起的吗？创建子进程的过程不理解，肯定是子进程引起的。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的哈。
https://github.com/nodejs/node/blob/master/lib/internal/cluster/primary.js
这里是调用 cluster.fork 的地方，这部分会调用createWorkerProcess创建子进程，这个方法中会调用 fork 方法，而 fork 在文件头部有申明const { fork } = require('child_process');因此最终会将启动参数命令行传递给 child_process 的 fork 方法来启动一个新的进程，再细致点，你可以看到它会把启动相关的参数都传递给 fork 这个函数。具体可以看下这个方法的官方介绍 https://nodejs.org/api/child_process.html#child_process_child_process_fork_modulepath_args_options 。

##### **杰：
> 老师，请问pm2的RPC通信连接和cluster的IPC主子通信有什么联系

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; RPC 通信其实是一种 IPC 通信方式。但是对于 PM2 和 cluster 模式来说这两者其实没什么关联性。最终目的都是实现消息的传递，只是 PM2 利用 RPC 命令行指令，而 cluster 应用 IPC 传递进程间的请求消息。

##### lastbee：
> cluster.js 为啥执行三次

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我这里没有非常理解你的问题点，这里说的 3 次是指我们源代码中的 cluster.js 这个文件吗，还是 Node.js的 cluster.js 这个模块。这2部分都没有执行3次的说法呢。前者启动后，就执行了一次。后者被require以后，并没有执行多次，而是调用了2次 cluster.fork()创建了2个子进程。不知道是专栏哪部份内容导致了误解，你可以补充下专栏的说明部分。

##### **6400：
> 老师，请问error_file和out_file是在build阶段通过shell脚本写入到项目目录吗？没太理解这个文件创建的时机

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是两个文件首先要创建目录，专栏里面说的要先创建好相应的目录，PM2在写日志的时候会直接在目录下写，并不会先创建目录。比如你用 fs 去写文件时，你目录不存在会直接报错，所以在使用 fs.write 时你要先创建好目录，PM2 也是一样的，只是他用的是文件流。创建时机就是你在上线这个项目之前，必须要创建好这个日志目录。

