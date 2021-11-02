<p data-nodeid="92145" class="">今天这一讲我会带你学习 Serverless 应用中的依赖管理。</p>
<p data-nodeid="92146">在前面的内容中，我们的 Serverless 应用代码都是独立的函数，不涉及其他依赖（在 Serverless 应用中，依赖可以分为两种：通过具体编程语言的包管理工具，如 npm、pip 等安装的包；通过 apt 等工具安装的程序运行时需要的软件包）。而在实际进行应用开发时，大部分情况下都会有第三方依赖。比如你要用 Node.js 操作 MySQL 时，肯定要用到操作数据库的依赖包（如 node-mysql2），然后你很快就写出如下的函数代码：</p>
<pre class="lang-javascript" data-nodeid="92147"><code data-language="javascript"><span class="hljs-keyword">const</span> mysql = <span class="hljs-built_in">require</span>(<span class="hljs-string">'mysql2/promise'</span>);
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">main</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-comment">// 获取数据库连接</span>
  <span class="hljs-keyword">const</span> connection = <span class="hljs-keyword">await</span> mysql.createConnection({<span class="hljs-attr">host</span>:<span class="hljs-string">'localhost'</span>, <span class="hljs-attr">user</span>: <span class="hljs-string">'root'</span>, <span class="hljs-attr">database</span>: <span class="hljs-string">'test'</span>});
  <span class="hljs-comment">// 查询数据库</span>
  <span class="hljs-keyword">const</span> [rows, fields] = <span class="hljs-keyword">await</span> connection.execute(<span class="hljs-string">'SELECT * FROM `table` WHERE `name` = ? AND `age` &gt; ?'</span>, [<span class="hljs-string">'Morty'</span>, <span class="hljs-number">14</span>]);
  <span class="hljs-keyword">return</span> rows;
}
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">event, context, callback</span>) </span>{ 
  main()
    .then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> callback(<span class="hljs-literal">null</span>, res))
    .catch(<span class="hljs-function"><span class="hljs-params">error</span> =&gt;</span> callback(error));
};
</code></pre>
<p data-nodeid="92148"><strong data-nodeid="92241">但问题来了：怎么安装 node-mysql2 这个包？</strong> 团队的小伙伴经常会问我类似的问题，如果你之前没有 Serverless 开发的经验，可能不知道怎么下手，如果有一定的经验，可能会遇到安装依赖报错，或依赖体积过大导致无法部署应用的问题。</p>
<p data-nodeid="92149">总的来说，怎么在 Serverless 应用中安装和管理依赖，一直都是困扰 Serverless 初学者的难题，所以我准备了这样一节课。这一讲我会从为什么安装依赖这么难、如何安装依赖这两个角度出发，并以 Node.js、Python 和 Java 为例，为你详细介绍 Serverless 中的依赖管理，让你真正学懂、学会。</p>
<h3 data-nodeid="92150">为什么安装依赖很困难</h3>
<p data-nodeid="92151">我觉得 Serverless 应用的依赖安装很困难，主要是因为它运行在 FaaS 平台上，而 FaaS 平台的运行环境由云厂商提供且预制，开发者只能进行有限的定制。对于传统应用，你可以直接登录服务器去安装应用运行时需要的依赖和软件，但在 Serverless 中就不行了，你都没有服务器了，更不用说登录服务器执行命令。</p>
<p data-nodeid="92152">而我说过，Serverless 应用是由函数组成的，函数在被触发执行时会生成函数实例，所以 Serverless 应用的依赖，本质上就是每个函数代码的依赖。</p>
<p data-nodeid="92153"><img src="https://s0.lgstatic.com/i/image2/M01/04/98/Cip5yF_z5MaAbj6SAAHFwPqDRvM991.png" alt="Drawing 0.png" data-nodeid="92248"></p>
<p data-nodeid="92154"><strong data-nodeid="92253">函数实例的实体就是容器，</strong> 容器的实现方案可以是 Docker、CoreOS rkt 甚至 LXC 等。FaaS 通过容器来隔离每个函数实例，也通过容器实现函数运行时的内存和 CPU 限制。比如你给函数分配 128MB 的内存，则函数实例对应的容器内存资源就只有 128MB。</p>
<p data-nodeid="92155">在容器内最主要的就是运行环境。运行环境包含编程语言、内置模块、FaaS 内置依赖和函数代码几个部分：</p>
<ul data-nodeid="92156">
<li data-nodeid="92157">
<p data-nodeid="92158">编程语言是你创建函数时指定的某个具体版本的编程语言，由 FaaS 平台提供（你在创建函数时必须指定编程语言）；</p>
</li>
<li data-nodeid="92159">
<p data-nodeid="92160">内置模块就是该编程语言的一些内置模块，比如 Node.js 中的 fs、http 等；</p>
</li>
<li data-nodeid="92161">
<p data-nodeid="92162">此外为了让开发者使用更方便，FaaS 平台一般还会默认安装一些依赖，比如函数计算的 Node.js 运行环境就默认提供了 <a href="https://github.com/aliyun-UED/aliyun-sdk-js" data-nodeid="92260">aliyun-sdk</a>、<a href="https://github.com/peterbraden/node-opencv" data-nodeid="92264">opencv</a> 等模块；</p>
</li>
<li data-nodeid="92163">
<p data-nodeid="92164">函数代码就是你的 Serverless 应用代码了，函数实例创建时，会从存储服务中将你的代码下载下来并加载到运行环境中。</p>
</li>
</ul>
<p data-nodeid="92165"><strong data-nodeid="92271">在整个函数实例中，你能控制的只有函数代码了。</strong> 所以如果函数代码需要安装依赖，实现方案就是将依赖安装到应用内部，将依赖和代码一并打包部署到 FaaS 平台中。</p>
<p data-nodeid="92166">虽然说起来简单，但实践起来往往并不容易，主要是因为几点：</p>
<ul data-nodeid="92167">
<li data-nodeid="92168">
<p data-nodeid="92169">大多编程语言的依赖通常安装在全局系统目录，比如 pip、maven 等工具会将依赖安装在项目之外的系统目录中；</p>
</li>
<li data-nodeid="92170">
<p data-nodeid="92171">安装依赖过程中可能涉及代码编译，环境不统一会导致编译产物有差异；</p>
</li>
<li data-nodeid="92172">
<p data-nodeid="92173">系统依赖通常不可移植，应用运行时依赖一些系统级别的动态链接库和软件。</p>
</li>
</ul>
<p data-nodeid="92174">接下来我就以一些实际安装依赖的示例带你了解如何去解决这上述问题。</p>
<h3 data-nodeid="92175">应该如何安装依赖</h3>
<p data-nodeid="92176">由于不同编程语言的包管理方式不同，所以依赖安装的方式也不尽相同。我会以<strong data-nodeid="92287">Java</strong>、<strong data-nodeid="92288">Node.js、Python</strong>这三种典型的编程语言带你了解 Serverless 应用如何安装依赖（这三种语言安装依赖的难度依次递增），即使你主要用其中某一种语言，我也希望你可以多了解不同语言的依赖处理，这样可以让你有更多的启发。</p>
<h4 data-nodeid="92177">如何为 Java 应用安装依赖</h4>
<p data-nodeid="92178">对于 Java 这种编译型语言，安装依赖反而更简单。因为 Java 应用部署前有个编译过程，编译后的 jar 包，就已经包含了应用代码及依赖，可以直接在 JVM 虚拟机中执行。所以在 FaaS 中部署的 Java 应用，是编译后的 jar 包，而不是原始代码。</p>
<p data-nodeid="92179"><strong data-nodeid="92294">虽然部署 jar 包不用关心依赖了，但这也带来了问题：部署前需要先编译。</strong></p>
<p data-nodeid="92180">通常我们本地开发时，Java 的依赖都是安装在本机的 $M2_HOME/repository 目录下，编译时会从根据本机的 JDK 版本以及本机的 repository 去编译代码。如果有多个同学同时开发部署一个应用，大家电脑上安装的 JDK 版本或依赖包可能不完全相同，这会导致每个人编译出的 jar 包不同，甚至导致部署到 FaaS 上的 jar 包无法正常运行。</p>
<p data-nodeid="92181">要解决编译环境的问题，就要有一个统一的构建机了，不让开发同学直接编译部署 Java 应用。</p>
<p data-nodeid="92182"><img src="https://s0.lgstatic.com/i/image/M00/8C/C1/CgqCHl_z5OCAZTJ3AAKzw8wyRJo086.png" alt="Drawing 1.png" data-nodeid="92301"></p>
<p data-nodeid="92183">如图所示，我们开发 Java 时可以通过 maven 的 pom.xml 来定义代码的依赖。部署 Java 应用时，需要将业务代码和 pom.xml 都上传到构建机，在构建机上统一安装依赖编译代码，得到一个可执行的 jar 包，最后再将 jar 包部署到 FaaS 平台。</p>
<p data-nodeid="92184">Java 是编译型语言，只要统一了编译环境，就不用关心依赖问题了。那对于 Node.js 这种解释型语言，应该怎么安装管理依赖呢？</p>
<h4 data-nodeid="92185">如何为 Node.js 应用安装依赖</h4>
<p data-nodeid="92186">在 Node.js 中，依赖分为全局依赖和项目依赖，全局依赖安装在系统目录中，项目依赖安装在项目目录中。项目依赖可以通过 package.json 来管理，在代码中你可以通过 require 方法来引入某个依赖，依赖的查找路径是先查询项目目录，再查询系统目录。你可以通过 module.paths 查看依赖查找的路径：</p>
<pre class="lang-powershell" data-nodeid="92187"><code data-language="powershell">&gt; module.paths
[
  <span class="hljs-string">'/Users/serverless/node_modules'</span>,
  <span class="hljs-string">'/Users/node_modules'</span>,
  <span class="hljs-string">'/node_modules'</span>,
  <span class="hljs-string">'/Users/serverless/.node_modules'</span>,
  <span class="hljs-string">'/Users/serverless/.node_libraries'</span>,
  <span class="hljs-string">'/usr/local/node/v15.3.0/lib/node'</span>
]
</code></pre>
<p data-nodeid="92188">所以得益于 Node.js 的包管理机制，要让运行在 FaaS 中的 Node.js 代码能够找到依赖，解决办法就是避免使用全局依赖，将所有依赖都安装到项目的 node_modules 中，然后将 node_modules 和代码一同部署到 FaaS。</p>
<p data-nodeid="92189">对于 Node.js 来说，除了可以直接安装在 node_moduels 中的 JS 依赖外，可能你还会使用 C++ 来编写一些 Node.js 扩展，这些 C++ 的扩展应该怎么安装呢？这时就需要你先将 C++ 代码编译为 .node 文件，然后再将编译产物和代码一同部署到 FaaS。</p>
<p data-nodeid="92190">为了方便你的理解，我编写了一个示例程序<a href="https://github.com/nodejh/serverless-class/tree/master/06" data-nodeid="92317">在 FaaS 中使用 Node.js C++ 扩展</a>，你可以参考一下。<strong data-nodeid="92323">这里尤其需要注意的是，</strong> 使用 C++ 编译后的可执行文件，需要与 FaaS 平台的运行环境兼容，不然无法运行。所以跟 Java 一样，编译 C++ 扩展时，也需要使用统一的构建机，并且这里构建机的环境还需要与 FaaS 平台的运行环境一致。</p>
<p data-nodeid="92191">当然，还有一些使用 Node.js 的同学会遇到依赖包体积太大，导致函数无法部署的问题。这是由于 FaaS 平台一般都会对代码体积有限制（比如函数计算限制是 100MB）。FaaS 对代码体积的限制，本质上是为了提升函数性能，因为 FaaS 创建函数实例时需要从文件存储中下载代码，代码体积过大则耗时越长，会影响函数启动速度。<strong data-nodeid="92331">在 Node.js 中，代码体积问题尤为常见，</strong> 因为 Node.js 项目的 node_moduls 体积经常很大。</p>
<p data-nodeid="92192">所以，虽然 Node.js 的 npm 生态为我们编程带来了便利，但 node_modules 嵌套层级深、冗余代码多，模块中还包含很多测试用例和源代码，这很容易导致代码体积快速膨胀。可能大部分时候，你引入一个新的依赖模块，真正需要执行的代码就几行，但包的体积却有几 MB。部署函数到 FaaS 上时，又需要部署 node_modules，造成了不必要的资源浪费。<strong data-nodeid="92340">这时我们就要想办法减小模块体积。</strong></p>
<p data-nodeid="92193">如何精简呢？你可以参考 Java 等编译型语言的做法，对 Node.js 代码也进行编译。在编译过程中，分析代码的依赖，只将需要使用到的模块和需要引入的函数构建打包，丢弃无用的代码，或者对代码进行压缩。这个过程其实就跟现在前端使用 webpack 构建 React.js 等框架一样。要实现 Node.js 代码的编译，也可以利用 webpack 来实现，<a href="https://github.com/vercel" data-nodeid="92344">Vercel</a> 就基于 webpack 开发了一个名为 <a href="https://github.com/vercel/ncc" data-nodeid="92348">ncc</a> 的工具。使用 ncc 可以把复杂的 Node.js 项目编译为单个文件，并且去掉不必要的依赖，在很大程度上减小代码体积。</p>
<p data-nodeid="92194">总的来说，Node.js 安装依赖比较方便，原因在于其依赖可以很轻松地安装在项目目录中，那对于 Python 这种依赖都安装在系统目录中的编程语言，你要怎么安装依赖呢？</p>
<h4 data-nodeid="92195">如何为 Python 应用安装依赖</h4>
<p data-nodeid="92196">pip 是当前 Python 中最流行的包管理工具，所以我们通常会使用 pip 来安装依赖。<strong data-nodeid="92357">但给 Python 项目安装依赖比较麻烦的地方就在于：</strong> 使用 pip 安装的依赖通常会散落在系统的各个文件中，比如 Python 类库和动态链接库会放在 /usr/lib/python/site-packages 中，而可执行文件则会存放在 /usr/bin 中。这样想要将所有文件都放在同一个目录中就会比较麻烦。</p>
<p data-nodeid="92197">pip 提供了多种把依赖安装在指定目录中的方案，但都存在一些问题，远没有 Node.js 的 node_modules 那么方便。下面我就分别举例说明。</p>
<ul data-nodeid="92198">
<li data-nodeid="92199">
<p data-nodeid="92200"><strong data-nodeid="92364">方法一：使用 --install-option 参数</strong></p>
</li>
</ul>
<p data-nodeid="92201">pip 提供了 --install-option 参数，可以让你精确控制某种类型的依赖安装路径。 如表格所示，--install-option 参数有多个可选项：</p>
<p data-nodeid="92202"><img src="https://s0.lgstatic.com/i/image2/M01/04/B7/Cip5yF_1Ve-AWbO8AALiP2fR8-E458.png" alt="Lark20210106-141708.png" data-nodeid="92368"></p>
<p data-nodeid="92203">其中 --install-lib 是指将所有模块都安装到指定目录。例如：</p>
<pre class="lang-powershell" data-nodeid="92204"><code data-language="powershell"><span class="hljs-comment"># 将模块安装到当面目录</span>
<span class="hljs-variable">$</span> pip install -<span class="hljs-literal">-install</span><span class="hljs-literal">-option</span>=<span class="hljs-string">"--install-lib=<span class="hljs-variable">$</span>(pwd)"</span> PyMySQL
</code></pre>
<p data-nodeid="92205">此外你也可以使用&nbsp;--install-option="--prefix=$(pwd)" 将依赖安装在当前目录的 lib/python3.7/site-packages 子目录下，这样更方便依赖的管理。</p>
<p data-nodeid="92206">但 --install-option 有个很大的缺点，就是安装过程中需要根据源码重新构建依赖。如果某个依赖包没有源码，就无法使用了，并且 --install-option 的参数设置也比较复杂。</p>
<p data-nodeid="92207">那有没有简单一点的方式呢？也有，那就是使用 --target&nbsp;参数。</p>
<ul data-nodeid="92208">
<li data-nodeid="92209">
<p data-nodeid="92210"><strong data-nodeid="92380">方法二：--target&nbsp;参数</strong></p>
</li>
</ul>
<p data-nodeid="92211">--targe 参数可以将依赖直接安装到当前目录，不会产生&nbsp;lib/python3.7/site-packages&nbsp;子目录解构。这个方法比较简单，适合依赖较少的情况。</p>
<ul data-nodeid="92212">
<li data-nodeid="92213">
<p data-nodeid="92214"><strong data-nodeid="92385">方法三：使用 virtualenv</strong></p>
</li>
</ul>
<p data-nodeid="92215">使用 vitualenv 可以将 python 包安装到一个独立的虚拟环境中，这样你就可以在这个虚拟环境中为代码安装所有依赖。形态上你的代码和依赖就完全在同一个目录中了，然后你就可以将代码和依赖一同部署到 FaaS。</p>
<pre class="lang-shell" data-nodeid="92216"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> pip install virtualenv</span>
<span class="hljs-meta">$</span><span class="bash"> virtualenv path/to/my/virtual-env</span>
<span class="hljs-meta">$</span><span class="bash"> <span class="hljs-built_in">source</span> path/to/my/virtual-env/bin/activate</span>
<span class="hljs-meta">$</span><span class="bash"> pip install PyMySQL</span>
</code></pre>
<p data-nodeid="92217">使用 vitualenv 的好处是不会污染全局环境，并且也会把包管理相关的工具都本地化，缺点就是会增加包的体积。</p>
<p data-nodeid="92218">其实对 Python 来说，只是把包安装到了当前目录还不够，你还需要修改 Python 解析依赖的路径。你可以通过 sys.path 查看 Python 解析依赖的路径，大概是下面这样：</p>
<pre class="lang-shell" data-nodeid="92219"><code data-language="shell"><span class="hljs-meta">&gt;</span><span class="bash">&gt;&gt; import sys</span>
<span class="hljs-meta">&gt;</span><span class="bash">&gt;&gt; <span class="hljs-built_in">print</span>(<span class="hljs-string">'\n'</span>.join(sys.path))</span>
''
/usr/lib/python/3.7
/usr/lib/python/3.7/lib-dynload
/usr/lib/Python/3.7/lib/python/site-packages
/usr/local/lib/python3.7/site-packages
</code></pre>
<p data-nodeid="92220">如果你的依赖直接安装在当前目录还好，因为 sys.path 包含了当前目录，代码中可以直接引入依赖。但通常我们并不会将依赖安装在当前目录，这样会导致当前目录文件十分混乱，而是将其安装在当前目录的 lib/python3.7/site-packages 子目录下。<strong data-nodeid="92395">这时就需要你在代码中修改 sys.path 了。</strong><br>
你需要在程序的开头加上这样的代码：</p>
<pre class="lang-python" data-nodeid="92221"><code data-language="python"><span class="hljs-comment"># index.py</span>
<span class="hljs-comment"># 将 lib/ 目录添加到 sys.path</span>
<span class="hljs-keyword">import</span> sys
<span class="hljs-keyword">import</span> os
<span class="hljs-keyword">try</span>:
    <span class="hljs-comment"># 检查 sys.path 中是否存在 lib/ 目录</span>
    sys.path.index(os.getcwd() + <span class="hljs-string">'/lib/python3.7/site-packages'</span>)
<span class="hljs-keyword">except</span> ValueError:
    <span class="hljs-comment"># 如果 lib/ 目录不存在 sys.path 中，则将其添加到 sys.path 数组的最前面</span>
    sys.path.insert(<span class="hljs-number">0</span>, os.getcwd() + <span class="hljs-string">'/lib/python3.7/site-packages'</span>)
<span class="hljs-keyword">import</span> pymysql.cursors
</code></pre>
<p data-nodeid="92222">将子目录加入 sys.path 后，你才能使用其中的依赖模块。</p>
<h3 data-nodeid="92223">总结</h3>
<p data-nodeid="92224">由于不同编程语言包管理机制不同，安装依赖的方式也不尽相同，<strong data-nodeid="92402">但本质上，都是需要将依赖安装到应用项目中，并且随项目一起部署到 FaaS 平台。</strong></p>
<p data-nodeid="92225"><img src="https://s0.lgstatic.com/i/image/M00/8C/E1/CgqCHl_1VaKAfRq1AAFoIplyjVs354.png" alt="Lark20210106-141448.png" data-nodeid="92405"></p>
<p data-nodeid="99353" class="te-preview-highlight">当然，如果你已经开始使用一些开发框架，你可能已经发现开发框架也在解决依赖安装问题，让用户尽可能更低成本完成应用开发。关于这一讲，我想要强调这几个点：</p>














<ul data-nodeid="92227">
<li data-nodeid="92228">
<p data-nodeid="92229">Serverless 应用的代码依赖和系统依赖都需要安装在项目中，并和应用代码一起部署到 FaaS 平台；</p>
</li>
<li data-nodeid="92230">
<p data-nodeid="92231">FaaS 对代码体积大小有限制，所以最好要精简依赖体积；</p>
</li>
<li data-nodeid="92232">
<p data-nodeid="92233">如果代码或依赖需要编译，则编译环境需要和 FaaS 运行环境兼容，不然编译后的产物可能无法运行。</p>
</li>
</ul>
<p data-nodeid="92234" class="">这节课我留给你的作业是：编写并部署一个需要安装依赖的函数代码。希望今天的内容能让你有所收获，我们下一讲见。</p>

---

### 精选评论

##### **3917：
> FAAS厂商应该支持定制镜像，这样可以提前将依赖打到镜像中，减小Serverless体积，加快启动速度

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，下一讲《07｜运行时：使用自定义运行时支持自定义编程语言》我就会讲到如何使用自定义镜像创建函数，以及打包依赖。

##### **坤：
> 您好，既然serverless底层也是容器方案，那么在编排层面如何保证容器毫秒级别的启动速度呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 毫秒级别的启动速度通常是指热启动。函数冷启动时，就需要启动容器，通常容器的极限启动时间为200ms，因此函数冷启动时间可能就需要几百毫秒。当第一次函数冷启动后，在一段时间内，Serverless FaaS 平台会将函数实例（包含容器及运行环境）保留一段时间，这样后面的请求就会复用上一次的容器，而不用重新启动容器了，这个过程就是热启动，热启动通常就是毫秒级。

