<p>我们在上一课时中，将 demo-provider 和 demo-webapp 接入 SkyWalking Agent 的时候，只需要在 VM options 中添加下面这一行配置即可：</p>
<pre><code data-language="js" class="lang-js">-javaagent:<span class="hljs-regexp">/path/</span>to/skywalking-agent.jar&nbsp;\
-Dskywalking_config=<span class="hljs-regexp">/path/</span>to/agent.config
</code></pre>
<p>并没有修改任何一行 Java 代码。这里便使用到了 Java Agent 技术，本课时我们将对 Java Agent 技术进行简单介绍并通过示例演示 Java Agent 技术的基本使用。</p>
<h3>什么是 Java Agent</h3>
<p>Java Agent 是从 JDK1.5 开始引入的，算是一个比较老的技术了。作为 Java 的开发工程师，我们常用的命令之一就是 java 命令，而 Java Agent 本身就是 java 命令的一个参数（即 -javaagent）。正如上一课时接入 SkyWalking Agent 那样，-javaagent 参数之后需要指定一个 jar 包，这个 jar 包需要同时满足下面两个条件：</p>
<ol>
<li>在 META-INF 目录下的 MANIFEST.MF 文件中必须指定 premain-class 配置项。</li>
<li>premain-class 配置项指定的类必须提供了 premain() 方法。</li>
</ol>
<p>在 Java 虚拟机启动时，执行 main() 函数之前，虚拟机会先找到 -javaagent 命令指定 jar 包，然后执行 premain-class 中的 premain() 方法。用一句概括其功能的话就是：main() 函数之前的一个拦截器。</p>
<p>使用 Java Agent 的步骤大致如下：</p>
<p>1.&nbsp; 定义一个 MANIFEST.MF 文件，在其中添加 premain-class 配置项。</p>
<p>2.&nbsp; 创建 &nbsp;premain-class 配置项指定的类，并在其中实现 premain() 方法，方法签名如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">premain</span><span class="hljs-params">(String&nbsp;agentArgs,&nbsp;Instrumentation&nbsp;inst)</span></span>{
&nbsp;&nbsp;&nbsp;...&nbsp;
}
</code></pre>
<p>3.&nbsp; 将 MANIFEST.MF 文件和 &nbsp;premain-class 指定的类一起打包成一个 jar 包。</p>
<p>4.&nbsp; 使用 -javaagent 指定该 jar 包的路径即可执行其中的 premain() 方法。</p>
<h3>Java Agent 示例</h3>
<p>首先，我们创建一个最基本的 Maven 项目，然后创建 TestAgent.java 这一个类，项目的整体结构如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/73/51/Cgq2xl5p4auAbrrvAACRfWvGnT4223.png" alt=""></p>
<p>TestAgent 中提供了 premain() 方法的实现，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">TestAgent</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">premain</span><span class="hljs-params">(String&nbsp;agentArgs,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Instrumentation&nbsp;inst)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"this&nbsp;is&nbsp;a&nbsp;java&nbsp;agent&nbsp;with&nbsp;two&nbsp;args"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"参数:"</span>&nbsp;+&nbsp;agentArgs&nbsp;+&nbsp;<span class="hljs-string">"\n"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">premain</span><span class="hljs-params">(String&nbsp;agentArgs)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"this&nbsp;is&nbsp;a&nbsp;java&nbsp;agent&nbsp;only&nbsp;one&nbsp;args"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"参数:"</span>&nbsp;+&nbsp;agentArgs&nbsp;+&nbsp;<span class="hljs-string">"\n"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>premain() 方法有两个重载，如下所示，如果两个重载同时存在，【1】将会被忽略，只执行【2】：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">premain</span><span class="hljs-params">(String&nbsp;agentArgs)</span>&nbsp;[1]
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">premain</span><span class="hljs-params">(String&nbsp;agentArgs,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Instrumentation&nbsp;inst)</span></span>;&nbsp;[<span class="hljs-number">2</span>]
</code></pre>
<p>代码中有两个参数需要我们注意。</p>
<ul>
<li><strong>agentArgs 参数</strong>：-javaagent 命令携带的参数。在前面介绍 SkyWalking Agent 接入时提到，agent.service_name 这个配置项的默认值有三种覆盖方式，其中，使用探针配置进行覆盖，探针配置的值就是通过该参数传入的。</li>
<li><strong>inst 参数</strong>：java.lang.instrumen.Instrumentation 是 Instrumention 包中定义的一个接口，它提供了操作类定义的相关方法。</li>
</ul>
<p>确定 premain() 方法的两个重载优先级的逻辑在 sun.instrument.InstrumentationImpl.java 中实现，相关代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">loadClassAndCallPremain</span><span class="hljs-params">(String&nbsp;&nbsp;String&nbsp;&nbsp;optionsString)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Throwable&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;loadClassAndStartAgent(&nbsp;classname,&nbsp;<span class="hljs-string">"premain"</span>,&nbsp;optionsString&nbsp;);
}

<span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">loadClassAndStartAgent</span><span class="hljs-params">(String&nbsp;&nbsp;classname,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;&nbsp;methodname,&nbsp;String&nbsp;&nbsp;optionsString)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Throwable&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;省略变量定义,下面省略&nbsp;try/catch代码块</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;查找两个参数的&nbsp;premain()方法</span>
&nbsp;&nbsp;&nbsp;&nbsp;m&nbsp;=&nbsp;javaAgentClass.getDeclaredMethod(&nbsp;methodname,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;Class&lt;?&gt;[]&nbsp;{String<span class="hljs-class">.<span class="hljs-keyword">class</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-title">java</span>.<span class="hljs-title">lang</span>.<span class="hljs-title">instrument</span>.<span class="hljs-title">Instrumentation</span>.<span class="hljs-title">class</span>})</span>;
&nbsp;&nbsp;&nbsp;&nbsp;twoArgAgent&nbsp;=&nbsp;<span class="hljs-keyword">true</span>;

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(m&nbsp;==&nbsp;<span class="hljs-keyword">null</span>)&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;查找一个参数的&nbsp;premain()方法</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m&nbsp;=&nbsp;javaAgentClass.getDeclaredMethod(methodname,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;Class&lt;?&gt;[]&nbsp;{&nbsp;String<span class="hljs-class">.<span class="hljs-keyword">class</span>&nbsp;})</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;...<span class="hljs-comment">//&nbsp;省略其他查找&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;setAccessible(m,&nbsp;<span class="hljs-keyword">true</span>);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;调用查找到的&nbsp;premain()重载</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(twoArgAgent)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m.invoke(<span class="hljs-keyword">null</span>,&nbsp;<span class="hljs-keyword">new</span>&nbsp;Object[]&nbsp;{&nbsp;optionsString,&nbsp;<span class="hljs-keyword">this</span>&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m.invoke(<span class="hljs-keyword">null</span>,&nbsp;<span class="hljs-keyword">new</span>&nbsp;Object[]&nbsp;{&nbsp;optionsString&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;setAccessible(m,&nbsp;<span class="hljs-keyword">false</span>);
</code></pre>
<p>接下来还需要创建 MANIFEST.MF 文件并打包，这里我们直接使用 maven-assembly-plugin 打包插件来完成这两项功能。在 pom.xml 中引入 maven-assembly-plugin 插件并添加相应的配置，如下所示：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">plugin</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.maven.plugins<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>maven-assembly-plugin<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>2.4<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">configuration</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">appendAssemblyId</span>&gt;</span>false<span class="hljs-tag">&lt;/<span class="hljs-name">appendAssemblyId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;将TestAgent的所有依赖包都打到jar包中--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">descriptorRefs</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">descriptorRef</span>&gt;</span>jar-with-dependencies<span class="hljs-tag">&lt;/<span class="hljs-name">descriptorRef</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">descriptorRefs</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">archive</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;添加MANIFEST.MF中的各项配置--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">manifest</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;添加&nbsp;mplementation-*和Specification-*配置项--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">addDefaultImplementationEntries</span>&gt;</span>true
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">addDefaultImplementationEntries</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">addDefaultSpecificationEntries</span>&gt;</span>true
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">addDefaultSpecificationEntries</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">manifest</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;将&nbsp;premain-class&nbsp;配置项设置为com.xxx.TestAgent--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">manifestEntries</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">Premain-Class</span>&gt;</span>com.xxx.TestAgent<span class="hljs-tag">&lt;/<span class="hljs-name">Premain-Class</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">manifestEntries</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">archive</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">configuration</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">executions</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">execution</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;绑定到package生命周期阶段上&nbsp;--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">phase</span>&gt;</span>package<span class="hljs-tag">&lt;/<span class="hljs-name">phase</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">goals</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;绑定到package生命周期阶段上&nbsp;--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">goal</span>&gt;</span>single<span class="hljs-tag">&lt;/<span class="hljs-name">goal</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">goals</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">execution</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">executions</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">plugin</span>&gt;</span>
</code></pre>
<p>最后执行 maven 命令进行打包，如下：</p>
<pre><code data-language="java" class="lang-java">mvn&nbsp;<span class="hljs-keyword">package</span>&nbsp;-Dcheckstyle.skip&nbsp;-DskipTests
</code></pre>
<p>完成打包之后，我们可以解压 target 目录下的 test-agent.jar，在其 META-INF 目录下可以找到 MANIFEST.MF 文件，其内容如下：</p>
<pre><code data-language="js" class="lang-js">Manifest-Version:&nbsp;1.0
Implementation-Title:&nbsp;TestAgent
Premain-Class:&nbsp;com.xxx.TestAgent&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;关注这一项
Implementation-Versioclearn:&nbsp;1.0.0-SNAPSHOT
Archiver-Version:&nbsp;Plexus&nbsp;Archiver
Built-By:&nbsp;xxx
Specification-Title:&nbsp;TestAgent
Implementation-Vendor-Id:&nbsp;com.xxx
Created-By:&nbsp;Apache&nbsp;Maven&nbsp;3.6.0
Build-Jdk:&nbsp;1.8.0_191
Specification-Version:&nbsp;1.0.0-SNAPSHOT
</code></pre>
<p>到此为止，Java Agent 使用的 jar 包创建完成了。</p>
<p>下面再创建一个普通的 Maven 项目：TestMain，项目结构与 TestAgent 类似，如图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/73/51/CgpOIF5p4auARfNmAACdrDhFMHg352.png" alt=""></p>
<p>在 Main 这个类中定义了该项目的入口 main() 方法，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Main</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">1000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"TestMain&nbsp;Main!"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>在启动 TestMain 项目之前，需要在 VM options 中使用 -javaagent 命令指定前面创建的 test-agent.jar，如图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/73/51/Cgq2xl5p4ayACZVOAABtprd-NZA652.png" alt=""></p>
<p>启动 TestMain 之后得到了如下输出：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">this</span>&nbsp;is&nbsp;a&nbsp;java&nbsp;agent.
参数:option1=value2,option2=value2
TestMain&nbsp;Main!
</code></pre>
<h3>修改类实现</h3>
<p>Java Agent 可以实现的功能远不止添加一行日志这么简单，这里需要关注一下 premain() 方法中的第二个参数：Instrumentation。Instrumentation 位于 java.lang.instrument 包中，通过这个工具包，我们可以编写一个强大的 Java Agent 程序，用来动态替换或是修改某些类的定义。</p>
<p>下面先来简单介绍一下 Instrumentation 中的核心 API 方法：</p>
<ul>
<li><strong>addTransformer()/removeTransformer() 方法</strong>：注册/注销一个 ClassFileTransformer 类的实例，该 Transformer 会在类加载的时候被调用，可用于修改类定义。</li>
<li><strong>redefineClasses() 方法</strong>：该方法针对的是已经加载的类，它会对传入的类进行重新定义。</li>
<li>**getAllLoadedClasses()方法：**返回当前 JVM 已加载的所有类。</li>
<li><strong>getInitiatedClasses() 方法</strong>：返回当前 JVM 已经初始化的类。</li>
<li><strong>getObjectSize()方法</strong>：获取参数指定的对象的大小。</li>
</ul>
<p>下面我们通过一个示例演示 Instrumentation 如何与 Java Agent 配合修改类定义。首先我们提供一个普通的 Java 类：TestClass，其中提供一个 getNumber() 方法：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">TestClass</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;<span class="hljs-title">getNumber</span><span class="hljs-params">()</span>&nbsp;</span>{&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-number">1</span>;&nbsp;&nbsp;}
}
</code></pre>
<p>编译生成 TestClass.class 文件之后，我们将 getNumber() 方法返回值修改为 2，然后再次编译，并将此次得到的 class 文件重命名为 TestClass.class.2 文件，如图所示，我们得到两个 TestClass.class 文件：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/73/51/CgpOIF5p4ayAQ7QjAAAJJFmWXV4100.png" alt=""></p>
<p>之后将 TestClass.getNumber() 方法返回值改回 1 ，重新编译。</p>
<p>然后编写一个 main() 方法，新建一个 TestClass 对象并输出其 getNumber() 方法的返回值：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Main</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-keyword">new</span>&nbsp;TestClass().getNumber());
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>接下来编写 premain() 方法，并注册一个 Transformer 对象：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">TestAgent</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">premain</span><span class="hljs-params">(String&nbsp;agentArgs,&nbsp;Instrumentation&nbsp;inst)</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;注册一个&nbsp;Transformer，该&nbsp;Transformer在类加载时被调用</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inst.addTransformer(<span class="hljs-keyword">new</span>&nbsp;Transformer(),&nbsp;<span class="hljs-keyword">true</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;inst.retransformClasses(TestClass<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"premain&nbsp;done"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>Transformer &nbsp;实现了 ClassFileTransformer，其中的 transform() 方法实现可以修改加载到的类的定义，具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Transformer</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">ClassFileTransformer</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">byte</span>[]&nbsp;transform(ClassLoader&nbsp;l,&nbsp;String&nbsp;className,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Class&lt;?&gt;&nbsp;c,&nbsp;ProtectionDomain&nbsp;pd,&nbsp;<span class="hljs-keyword">byte</span>[]&nbsp;b)&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(!c.getSimpleName().equals(<span class="hljs-string">"TestClass"</span>))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">null</span>;&nbsp;<span class="hljs-comment">//&nbsp;只修改TestClass的定义</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;读取&nbsp;TestClass.class.2这个&nbsp;class文件，作为&nbsp;TestClass类的新定义</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;getBytesFromFile(<span class="hljs-string">"TestClass.class.2"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>之后还需要在 maven-assembly-plugin 插件中添加 Can-Retransform-Classes 参数：</p>
<pre><code data-language="html" class="lang-html">//&nbsp;省略其他配置
<span class="hljs-tag">&lt;<span class="hljs-name">manifestEntries</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">Premain-Class</span>&gt;</span>com.xxx.TestAgent<span class="hljs-tag">&lt;/<span class="hljs-name">Premain-Class</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">Can-Retransform-Classes</span>&gt;</span>true<span class="hljs-tag">&lt;/<span class="hljs-name">Can-Retransform-Classes</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">manifestEntries</span>&gt;</span>
</code></pre>
<p>最后，打包启动应用，得到的输出如下：</p>
<pre><code data-language="java" class="lang-java">premain&nbsp;done
2&nbsp;
#&nbsp;此输出说明，类TestClass的定义在加载时已经被&nbsp;Transformer替换成&nbsp;
#&nbsp;文件&nbsp;TestClass.class.2&nbsp;中的定义
</code></pre>
<h3>统计方法耗时</h3>
<p>介绍完 Java Agent 的基本使用流程之后，这里做一个简单的进阶示例，将 Java Agent 与 Byte Buddy 结合使用，统计 com.xxx 包下所有方法的耗时。</p>
<p>回到 TestAgent 项目，整个项目的结构不变，需要在 pom.xml 中添加 Byte Buddy 的依赖，如下所示：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>net.bytebuddy<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>byte-buddy<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.9.2<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>net.bytebuddy<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>byte-buddy-agent<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.9.2<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p>Byte Buddy 是一个开源 Java 库，其主要功能是帮助用户屏蔽字节码操作，以及复杂的 Instrumentation API 。Byte Buddy 提供了一套类型安全的 API 和注解，我们可以直接使用这些 API 和注解轻松实现复杂的字节码操作。另外，Byte Buddy 提供了针对 Java Agent 的额外 API，帮助开发人员在 Java Agent 场景轻松增强已有代码。在 下一课时中，我们将深入介绍 SkyWalking 涉及的 Byte Buddy 的 API，这里不做深入研究，只对此处涉及的 API 和注解做简单说明。</p>
<p>接下来对 TestAgent.java 进行修改，使用 Byte Buddy 提供的 API 拦截指定的类和方法，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">TestAgent</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">premain</span><span class="hljs-params">(String&nbsp;agentArgs,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Instrumentation&nbsp;inst)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;Byte&nbsp;Buddy会根据&nbsp;Transformer指定的规则进行拦截并增强代码</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AgentBuilder.Transformer&nbsp;transformer&nbsp;=&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;AgentBuilder.Transformer()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;DynamicType.Builder&lt;?&gt;&nbsp;transform(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DynamicType.Builder&lt;?&gt;&nbsp;builder,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TypeDescription&nbsp;typeDescription,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ClassLoader&nbsp;classLoader,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JavaModule&nbsp;<span class="hljs-keyword">module</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;method()指定哪些方法需要被拦截，ElementMatchers.any()表&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;示拦截所有方法</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;builder.method(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ElementMatchers.&lt;MethodDescription&gt;any())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;intercept()指明拦截上述方法的拦截器</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.intercept(MethodDelegation.to(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TimeInterceptor<span class="hljs-class">.<span class="hljs-keyword">class</span>))</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;Byte&nbsp;Buddy专门有个AgentBuilder来处理Java&nbsp;Agent的场景</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;AgentBuilder&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Default()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;根据包名前缀拦截类</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.type(ElementMatchers.nameStartsWith(<span class="hljs-string">"com.xxx"</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;拦截到的类由transformer处理</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.transform(transformer)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.installOn(inst);&nbsp;<span class="hljs-comment">//&nbsp;安装到&nbsp;Instrumentation</span>
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>当拦截到符合条件的类时，会交给我们的 AgentBuilder.Transformer 实现处理，当 Transformer 拦截到符合条件的方法时，会交给上面指定的 TimeInterceptor 处理。TimeInterceptor 的具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">TimeInterceptor</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@RuntimeType</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;Object&nbsp;<span class="hljs-title">intercept</span><span class="hljs-params">(@Origin&nbsp;Method&nbsp;method,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@SuperCall&nbsp;Callable&lt;?&gt;&nbsp;callable)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">long</span>&nbsp;start&nbsp;=&nbsp;System.currentTimeMillis();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;callable.call();&nbsp;<span class="hljs-comment">//&nbsp;执行原函数</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">finally</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(method.getName()&nbsp;+&nbsp;<span class="hljs-string">":"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;(System.currentTimeMillis()&nbsp;-&nbsp;start)&nbsp;+&nbsp;<span class="hljs-string">"ms"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>TimeInterceptor 就类似于 AOP 的环绕切面。这里通过 @SuperCall 注解注入的 Callable 实例可以调到被拦截的目标方法，需要注意的是，在通过 Callable 调用目标方法时，即使目标方法带参数，这里也不用显式的传递。这里 @Origin 注解注入了被拦截方法对应的 Method &nbsp;对象。</p>
<p>完成 TestAgent 的修改之后，执行如下命令，重新打包 test-agent.jar：</p>
<pre><code data-language="java" class="lang-java">mvn&nbsp;clean&nbsp;&nbsp;<span class="hljs-keyword">package</span>&nbsp;-Dcheckstyle.skip&nbsp;-DskipTests
</code></pre>
<p>打包完成，回到 TestMain 项目，所有代码和配置都无需修改，直接启动，会得到输出如下：</p>
<pre><code data-language="js" class="lang-js">TestMain&nbsp;Main!
main:<span class="hljs-number">1003</span>ms
</code></pre>
<h3>Attach API 基础</h3>
<p>在 Java 5 中，Java 开发者只能通过 Java Agent 中的 premain() 方法在 main() 方法执行之前进行一些操作，这种方式在一定程度上限制了灵活性。Java 6 针对这种状况做出了改进，提供了一个 agentmain() 方法，Java 开发者可以在 main() 方法执行以后执行 agentmain() 方法实现一些特殊功能。</p>
<p>agentmain() 方法同样有两个重载，它们的参数与 premain() 方法相同，而且前者优先级也是高于后者的：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">agentmain</span>&nbsp;<span class="hljs-params">(String&nbsp;agentArgs,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Instrumentation&nbsp;inst)</span></span>;[<span class="hljs-number">1</span>]

<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">agentmain</span>&nbsp;<span class="hljs-params">(String&nbsp;agentArgs)</span></span>;&nbsp;[<span class="hljs-number">2</span>]
</code></pre>
<p>agentmain() 方法主要用在 JVM Attach 工具中，Attach API 是 Java 的扩展 API，可以向目标 JVM “附着”（Attach）一个代理工具程序，而这个代理工具程序的入口就是 agentmain() 方法。</p>
<p>Attach API 中有 2 个核心类需要特别说明：</p>
<ul>
<li><strong>VirtualMachine</strong> 是对一个 Java 虚拟机的抽象，在 Attach 工具程序监控目标虚拟机的时候会用到该类。VirtualMachine 提供了 JVM 枚举、Attach、Detach 等基本操作。</li>
<li><strong>VirtualMachineDescriptor</strong> 是一个描述虚拟机的容器类，后面示例中会介绍它如何与 VirtualMachine 配合使用。</li>
</ul>
<p>下面的示例依然通过用类文件替换的方式修改 TestClass 这个类的返回值。首先，前文使用到的 TestClass，以及 TestClass.class.2 文件不变。main() 方法略作修改，其中每隔 1s 创建一个新的 TestClass 对象并输出 getNumber() 方法返回值，具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-keyword">new</span>&nbsp;TestClass().getNumber());
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(<span class="hljs-keyword">true</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">1000</span>);&nbsp;<span class="hljs-comment">//&nbsp;注意，这里是新建TestClass对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-keyword">new</span>&nbsp;TestClass().getNumber());
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>另外，将 TestAgent 中的 premain() 方法修改成 agentmain() 方法（除了名称变化，没有任何其他变化）。</p>
<p>需要再添加一个 AttachMain 类，其中会通过 Attach API 监听上面 main() 方法的启动，这里每隔 1s 检查一次所有的 Java 虚拟机，当发现有新的虚拟机出现的时候，就调用 attach() 方法，将 TestAgent 所在的 jar 包“附加”上去，“附加”成功之后，TestAgent 就会通过 Transformer 修改TestClass 类定义。AttachMain 的具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">AttachMain</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;List&lt;VirtualMachineDescriptor&gt;&nbsp;listBefore&nbsp;=&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VirtualMachine.list();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;jar&nbsp;=&nbsp;<span class="hljs-string">".../test-agent.jar"</span>;&nbsp;<span class="hljs-comment">//&nbsp;agentmain()方法所在jar包</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VirtualMachine&nbsp;vm&nbsp;=&nbsp;<span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;List&lt;VirtualMachineDescriptor&gt;&nbsp;listAfter&nbsp;=&nbsp;<span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(<span class="hljs-keyword">true</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;listAfter&nbsp;=&nbsp;VirtualMachine.list();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(VirtualMachineDescriptor&nbsp;vmd&nbsp;:&nbsp;listAfter)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(!listBefore.contains(vmd))&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;发现新的JVM</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vm&nbsp;=&nbsp;VirtualMachine.attach(vmd);&nbsp;<span class="hljs-comment">//&nbsp;attach到新JVM</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vm.loadAgent(jar);&nbsp;<span class="hljs-comment">//&nbsp;加载agentmain所在的jar包</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vm.detach();&nbsp;<span class="hljs-comment">//&nbsp;detach</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">1000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>在 pom.xml 文件中添加如下依赖，否则在编译的过程中会抛异常：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.sun<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>tools<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.8<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">scope</span>&gt;</span>system<span class="hljs-tag">&lt;/<span class="hljs-name">scope</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">systemPath</span>&gt;</span>${java.home}/../lib/tools.jar<span class="hljs-tag">&lt;/<span class="hljs-name">systemPath</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p>另外，还要在 MANIFEST.MF 文件中添加 Agent-Class 这一项， pom.xml 文件中具体配置如下：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">manifestEntries</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">Can-Retransform-Classes</span>&gt;</span>true<span class="hljs-tag">&lt;/<span class="hljs-name">Can-Retransform-Classes</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;指向agentmain()方法所在的类&nbsp;--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">Agent-Class</span>&gt;</span>com.xxx.TestAgent<span class="hljs-tag">&lt;/<span class="hljs-name">Agent-Class</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">manifestEntries</span>&gt;</span>
</code></pre>
<p>最后，我们先启动 AttachMain 类，然后通过命令行启动 Main ：</p>
<pre><code data-language="js" class="lang-js">java&nbsp;&nbsp;-cp&nbsp;./test-agent.jar&nbsp;com.xxx.attach.Main
-------
输出：
1
premain&nbsp;done&nbsp;#&nbsp;attach成功，Transformer会修改TestClass的定义
2&nbsp;&nbsp;#&nbsp;修改后的TestClass.getNumber()方法返回值为&nbsp;2
2
</code></pre>
<p>在 SkyWalking Agent 中并没有使用到 Attach API，但是作为 Java Agent 的扩展，还是希望你对其有所了解。</p>
<h3>总结</h3>
<p>本课时重点介绍了 Java Agent 技术的基础知识，然后通过 TestAgent 和 TestMain 两个示例项目简单展示了 Java Agent 技术的使用流程，之后通过 Java Agent 和 Byte Buddy 实现了统计方法耗时的功能，最后还简单介绍 Attach API 的相关功能并进行了示例演示。</p>
<p>希望你在课后可以自己多加练习。</p>

---

### 精选评论

##### **期：
> 不错，非常感谢老师的讲解！

##### **3723：
> 为什么不是使用最新版本的elk skywalking

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在开始写的时候，最新的版本6.3，其实 7.x 和 6.x 的核心原理基本相同的。以一人之力，追一个开源社区的更新，很快就追不上了

##### **富：
> 喜欢这种，有例有据

##### **6217：
> 请问一下，TestCase为什么要放到TestAgent这个项目下面呢，这样TestAgent不是依赖了业务class吗，或者TestAgent能够接入外部传入进来的参数，以此来动态指定retransformClass的类

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是个 Demo 示例，实践中会将接口单独封装到一个jar包，然后被外部依赖

##### **武：
> getBytesFromFile 这个方法是怎么实现的？<div><br></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 所有代码都在 https://github.com/xxxlxy2008/skywalking-demo

##### *宇：
> 课件里面没有代码啊！！！

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 可以关注拉勾教育公众号 咨询小助手获取代码

##### **生：
> 请问老师demo的代码可以在哪里下载呢？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 关注 拉勾教育 公众号 咨询小助手获取课件和代码

##### **办法：
> <div>getBytesFromFile("TestClass.class.2")</div><div>这是自定义方法吗？</div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，所有示例代码将在这里同步更新 https://github.com/xxxlxy2008/skywalking-demo

##### **3723：
> 现在es 7以上版本支持sky walking6.5

