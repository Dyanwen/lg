<p data-nodeid="2293" class="">本课时主要讲解“日志收集、分析过滤工具 Logstash 应用实战”。</p>


<h3 data-nodeid="9931" class="">Logstash 介绍与安装</h3>





<p data-nodeid="4">Logstash 是一款轻量级的、开源的日志收集处理框架，它可以方便地把分散的、多样化的日志搜集起来，并进行自定义过滤分析处理，然后传输到指定的位置，比如某个服务器或者文件。</p>
<p data-nodeid="5">Logstash 的理念很简单，从功能上来讲，它只做 3 件事情：</p>
<ul data-nodeid="6">
<li data-nodeid="7">
<p data-nodeid="8">input，数据收集；</p>
</li>
<li data-nodeid="9">
<p data-nodeid="10">filter，数据加工，比如过滤、修改等；</p>
</li>
<li data-nodeid="11">
<p data-nodeid="12">output，数据输出。</p>
</li>
</ul>
<p data-nodeid="13">由此可知，Logstash 实现的功能主要分为<strong data-nodeid="238">接收数据</strong>、<strong data-nodeid="239">解析过滤并转换数据</strong>、<strong data-nodeid="240">输出数据</strong>三个部分，这三个部分对应的插件依次是 input 插件、filter 插件、output 插件。其中，filter 插件是可选的，其他两个是必须插件，也就是说在一个完整的 Logstash 配置文件中，必须有 input 插件和 output 插件。</p>
<p data-nodeid="17461" class="">Logstash 安装非常简单，只需要下载解压即可，不过需要安装 Java 运行环境，即 JDK，你可以<a href="https://www.elastic.co/downloads/logstash" data-nodeid="17465">点击 Elastic 官网获取 Logstash 安装包</a>，这里下载的版本是 logstash-7.7.1.tar.gz。将下载下来的安装包直接解压到一个路径下即可完成安装，这里我将 logstash 安装到 nnmaster.cloud 主机（172.16.213.151）上，将 logstash 程序安装到 /usr/local 目录下。基本操作过程如下：</p>





<pre class="lang-dart" data-nodeid="52818"><code data-language="dart">[root<span class="hljs-meta">@logstashserver</span> ~]# tar -zxvf logstash<span class="hljs-number">-7.7</span><span class="hljs-number">.1</span>.tar.gz -C /usr/local
[root<span class="hljs-meta">@logstashserver</span> ~]# mv /usr/local/logstash<span class="hljs-number">-7.7</span><span class="hljs-number">.1</span> /usr/local/logstash
</code></pre>
























<p data-nodeid="16">这里我们将 logstash 安装到了 /usr/local 目录下。</p>
<h3 data-nodeid="61623" class="">如何编写 Logstash 配置文件</h3>






<p data-nodeid="18">Logstash 的配置文件在安装程序下的 config 子目录下，其中，jvm.options 是设置 JVM 内存资源的配置文件；logstash.yml 是 Logstash 全局属性配置文件，一般无须修改，另外还需要自己创建一个 Logstash 事件配置文件。这里重点介绍下 Logstash 事件配置文件的编写方法和使用方式。</p>
<p data-nodeid="19">在介绍 Logstash 配置之前，先来认识一下 Logstash 是如何实现输入和输出的。Logstash 提供了一个 shell 脚本 /usr/local/logstash/bin/logstash，可以方便快速地启动一个 Logstash 进程。在 Linux 命令行下，运行如下命令启动 Logstash 进程：</p>
<pre class="lang-dart" data-nodeid="78964"><code data-language="dart">[root<span class="hljs-meta">@logstashserver</span> ~]# cd /usr/local/logstash/
[root<span class="hljs-meta">@logstashserver</span> logstash]# bin/logstash -e <span class="hljs-string">'input{stdin{}} output{stdout{codec=&gt;rubydebug}}'</span>
</code></pre>












<p data-nodeid="21">首先解释下这条命令的含义：</p>
<ul data-nodeid="22">
<li data-nodeid="23">
<p data-nodeid="24">-e 代表执行的意思；</p>
</li>
<li data-nodeid="25">
<p data-nodeid="26">input 即输入的意思，其里面是输入的方式，这里选择了 stdin，也就是标准输入（从终端输入）；</p>
</li>
<li data-nodeid="27">
<p data-nodeid="28">output 即输出的意思，其里面是输出的方式，这里选择了 stdout，也就是标准输出（输出到终端），其中 codec 是个插件，表明格式，这里放在 stdout 中，表示输出的格式；rubydebug 是专门用来做测试的格式，一般用来在终端输出 JSON 格式。</p>
</li>
</ul>
<p data-nodeid="29">接着，在终端输入信息。这里我们输入 "Hello World"，按回车，马上就会有返回结果，内容如下：</p>
<pre class="lang-java" data-nodeid="84744"><code data-language="java">{
    <span class="hljs-string">"@timestamp"</span> =&gt; <span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">15</span>T10:<span class="hljs-number">08</span>:<span class="hljs-number">55.611</span>Z,
       <span class="hljs-string">"message"</span> =&gt; <span class="hljs-string">"Hello World"</span>,
          <span class="hljs-string">"host"</span> =&gt; <span class="hljs-string">"nnmaster.cloud"</span>,
      <span class="hljs-string">"@version"</span> =&gt; <span class="hljs-string">"1"</span>
}
</code></pre>




<p data-nodeid="87644" class="">这就是 Logstash 的输出格式，在输出内容中会给事件添加一些额外信息。比如 "@version""host""@timestamp" 都是新增的字段，而最重要的是 @timestamp，用来标记事件的发生时间。由于这个字段涉及 Logstash 内部流转，如果给一个字符串字段重命名为 @timestamp 的话，Logstash 就会直接报错。另外，也不能删除这个字段。</p>


<p data-nodeid="32">在 Logstash 的输出中，常见的字段还有 type，表示事件的唯一类型；tags 表示事件的某方面属性，我们可以随意给事件添加字段或者从事件里删除字段。在执行上面的命令后，可以看到，你输入什么内容，Logstash 就会按照上面的格式输出什么内容。使用 CTRL-C 命令可以退出运行的 Logstash 事件。</p>
<p data-nodeid="33">使用 -e 参数在命令行中指定配置是不常用的方式，但是如果 Logstash 需要配置更多规则的话，就必须把配置固化到文件里，这就是 Logstash 事件配置文件。如果把上面命令行执行的 Logstash 命令，写到一个配置文件 logstash-simple.conf 中，就变成如下的内容：</p>
<pre class="lang-js" data-nodeid="99183"><code data-language="js">input { 
stdin { }
}
output {
stdout { <span class="hljs-function"><span class="hljs-params">codec</span> =&gt;</span> rubydebug }
}
</code></pre>








<p data-nodeid="35">这就是最简单的 Logstash 事件配置文件。此时，还可以使用 Logstash 的 -f 参数来读取配置文件，然后启动 Logstash 进程，操作如下：</p>
<pre class="lang-dart" data-nodeid="119357"><code data-language="dart">[root<span class="hljs-meta">@logstashserver</span> logstash]# bin/logstash -f logstash-simple.conf
</code></pre>














<p data-nodeid="37">通过这种方式也可以启动 Logstash 进程，不过这种方式启动的进程是在前台运行的，若要放到后台运行，可通过 nohup 命令实现，操作如下：</p>
<pre class="lang-dart" data-nodeid="120798"><code data-language="dart">[root<span class="hljs-meta">@logstashserver</span> logstash]# nohup bin/logstash -f logstash-simple.conf &amp;
</code></pre>

<p data-nodeid="39">这样，Logstash 进程就放到后台运行了，在当前目录会生成一个 nohup.out 文件，可通过此文件查看 Logstash 进程的启动状态。</p>
<h3 data-nodeid="122239" class="">Logstash 输入插件（Input）</h3>

<p data-nodeid="41">Logstash 的输入插件主要用来接收数据，它支持多种数据源，常见的有<strong data-nodeid="340">读取文件</strong>、<strong data-nodeid="341">标准输入</strong>、<strong data-nodeid="342">读取网络数据</strong>等，这里分别介绍下每种接收数据源的配置方法。</p>
<h4 data-nodeid="123647" class="">1. Logstash 基本语法组成</h4>

<p data-nodeid="43">Logstash 配置文件由如下 3 部分组成，其中 input、output 部分是必须配置，filter 部分是可选配置，而 filter 就是过滤器插件，可以在这部分实现各种日志过滤功能。</p>
<pre class="lang-java" data-nodeid="153986"><code data-language="java">input {
输入插件
}
filter {
过滤匹配插件
}
output {
输出插件
}
</code></pre>






















<p data-nodeid="45">下面我将依次进行介绍。</p>
<h4 data-nodeid="155365" class="">2. Logstash 从文件读取数据</h4>

<p data-nodeid="47">Logstash 使用一个名为 filewatch 的 ruby gem 库来监听文件变化，并通过一个叫 .sincedb 的数据库文件来记录被监听的日志文件的读取进度（时间戳），该数据文件的默认路径在 &lt;path.data&gt;/plugins/inputs/file 下面，文件名类似于.sincedb_452905a167cf4509fd08acb964fdb20c，而 &lt;path.data&gt; 表示 Logstash 插件存储目录，默认是 LOGSTASH_HOME/data。</p>
<p data-nodeid="48">看下面一个事件配置文件：</p>
<pre class="lang-java" data-nodeid="163472"><code data-language="java">input {
    file {
        path =&gt; [<span class="hljs-string">"/var/log/secure"</span>]
        type =&gt; <span class="hljs-string">"system"</span>
        start_position =&gt; <span class="hljs-string">"beginning"</span>
    }
}
output {
    stdout{
        codec=&gt;rubydebug    
    }
}
</code></pre>






<p data-nodeid="164823" class="">这个配置是监听并接收本机的 /var/log/secure 文件内容，start_position 表示按时间戳记录的地方开始读取，如果没有时间戳则从头开始读取，有点类似 cat 命令。默认情况下，Logstash 会从文件的结束位置开始读取数据，也就是说 Logstash 进程会以类似 tail -f 命令的形式逐行获取数据。type 用来标记事件类型，通常会在输入区域通过 type 标记事件类型。</p>

<p data-nodeid="51">假定 /var/log/secure中输入的内容为如下信息：</p>
<pre class="lang-java" data-nodeid="170230"><code data-language="java">Jun <span class="hljs-number">16</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span>:<span class="hljs-number">52</span> nnmaster sshd[<span class="hljs-number">2854</span>]: Failed password <span class="hljs-keyword">for</span> root from <span class="hljs-number">172.16</span>.<span class="hljs-number">213.226</span> port <span class="hljs-number">46460</span> ssh2
</code></pre>




<p data-nodeid="53">那么经过 Logstash 后，会输出内容为如下 JSON 格式：</p>
<pre class="lang-java" data-nodeid="178336"><code data-language="java">{
       <span class="hljs-string">"message"</span> =&gt; <span class="hljs-string">"Jun 16 14:57:52 nnmaster sshd[2854]: Failed password for root from 172.16.213.226 port 46460 ssh2"</span>,
          <span class="hljs-string">"host"</span> =&gt; <span class="hljs-string">"nnmaster.cloud"</span>,
    <span class="hljs-string">"@timestamp"</span> =&gt; <span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">16</span>T06:<span class="hljs-number">57</span>:<span class="hljs-number">52.675</span>Z,
      <span class="hljs-string">"@version"</span> =&gt; <span class="hljs-string">"1"</span>,
          <span class="hljs-string">"path"</span> =&gt; <span class="hljs-string">"/var/log/secure"</span>,
          <span class="hljs-string">"type"</span> =&gt; <span class="hljs-string">"system"</span>
}
</code></pre>






<p data-nodeid="55">从输出可以看出，message 字段是真正的输出内容，将输入的信息原样输出了，最后还有一个 type 字段，这是在 input 中定义的一个事件类型，也被原样输出了，在后面将要介绍的过滤插件中会用到这个 type 字段。</p>
<h4 data-nodeid="179687" class="">3. Logstash 从标准输入读取数据</h4>

<p data-nodeid="57">stdin 是从标准输入获取信息，下面是一个关于 stdin 的事件配置文件：</p>
<pre class="lang-java" data-nodeid="184980"><code data-language="java">input{
    stdin{
        add_field=&gt;{<span class="hljs-string">"key"</span>=&gt;<span class="hljs-string">"ok"</span>}
        tags=&gt;[<span class="hljs-string">"add field"</span>]
        type=&gt;<span class="hljs-string">"mytype"</span>
    }
}
output {
    stdout{
        codec=&gt;rubydebug    
    }
}
</code></pre>




<p data-nodeid="59">如果输入 "hello world"，那么可以在终端看到如下输出信息：</p>
<pre class="lang-java" data-nodeid="192918"><code data-language="java">{
       <span class="hljs-string">"host"</span> =&gt; <span class="hljs-string">"nnmaster.cloud"</span>,
       <span class="hljs-string">"message"</span> =&gt; <span class="hljs-string">"hello world"</span>,
       <span class="hljs-string">"type"</span> =&gt; <span class="hljs-string">"mytype"</span>,
       <span class="hljs-string">"tags"</span> =&gt; [
       [<span class="hljs-number">0</span>] <span class="hljs-string">"add field"</span>
    ],
       <span class="hljs-string">"key"</span> =&gt; <span class="hljs-string">"ok"</span>,
       <span class="hljs-string">"@timestamp"</span> =&gt; <span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">15</span>T10:<span class="hljs-number">45</span>:<span class="hljs-number">56.419</span>Z,
       <span class="hljs-string">"@version"</span> =&gt; <span class="hljs-string">"1"</span>
} 
</code></pre>






<p data-nodeid="61">type 和 tags 是 Logstash 的两个特殊字段，type 一般会放在 input 中标记事件类型，tags 主要用于在事件中增加标签，以便在后续的处理流程中使用，主要用于 filter 或 output 阶段。</p>
<h4 data-nodeid="194241" class="">4. Logstash 从网络读取 TCP 数据</h4>

<p data-nodeid="63">下面的事件配置文件就是通过“LogStash::Inputs::TCP”和“LogStash::Filters::Grok”配合实现 syslog 功能的例子，这里使用了 logstash 的 TCP/UDP 插件读取网络数据：</p>
<pre class="lang-java" data-nodeid="199326"><code data-language="java">input {
  tcp {
    port =&gt; <span class="hljs-string">"5044"</span>
  }
}
filter {
  grok {
    match =&gt; { <span class="hljs-string">"message"</span> =&gt; <span class="hljs-string">"%{SYSLOGLINE}"</span> }
  }
}
output {
    stdout{
        codec=&gt;rubydebug
    }
}
</code></pre>




<p data-nodeid="200597">其中，5044 端口是 Logstash 启动的 TCP 监听端口。注意这里用到了日志过滤“LogStash::Filters::Grok”功能，下面马上会介绍。</p>
<p data-nodeid="200598">“LogStash::Inputs::TCP”最常见的用法就是结合 nc 命令导入旧数据。启动 Logstash 进程后，在另一个终端运行如下命令即可导入旧数据：</p>

<pre class="lang-dart" data-nodeid="218394"><code data-language="dart">[root<span class="hljs-meta">@kafkazk</span>1 app]# nc <span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.151</span>  <span class="hljs-number">5044</span>&lt; /<span class="hljs-keyword">var</span>/log/secure
</code></pre>














<p data-nodeid="67">通过这种方式，就把 /var/log/secure 的内容全部导入到 Logstash 中了，当 nc 命令结束时，数据也就导入完成了。</p>
<p data-nodeid="68">这里其实还可以将 Filebeat 收集到的日志直接导入到 Logstash，也就是在 Filebeat 的输出部分，做如下设置：</p>
<pre class="lang-dart" data-nodeid="223478"><code data-language="dart">output.logstash:
hosts: [<span class="hljs-string">"nnmaster.cloud:5044"</span>]
</code></pre>




<p data-nodeid="70">这样，Filebeat 就可以向 nnmaster.cloud 的 5044 端口发送数据，而 Logstash 就可以接收此数据，进而执行过滤、分析、输出等操作。</p>
<p data-nodeid="71">此时 Logstash 的 input 部分应该配置如下：</p>
<pre class="lang-java" data-nodeid="231104"><code data-language="java">input {
  beats {
   port =&gt; <span class="hljs-number">5044</span>
 }
}
</code></pre>






<p data-nodeid="73">注意，这里使用了 beats 主要用来接收从 Filebeat 发送过来的数据。</p>
<h3 data-nodeid="232375" class="">Logstash 编码插件（Codec）</h3>

<p data-nodeid="75">在前面介绍的例子中，其实我们已经用过编码插件 Codec 了，即 rubydebug，它是一种编码插件，一般只用在 Stdout 插件中，作为配置测试或者调试的工具。</p>
<p data-nodeid="233613" class=""><strong data-nodeid="233618">编码插件（Codec）</strong> 可以在 Logstash 中输入或输出时处理不同类型的数据，同时，还可以更好更方便地与其他自定义格式的数据产品共存，比如 fluent、netflow 等通用数据格式的其他产品。因此，Logstash 不只是一个 input → filter → output 的数据流，而是一个 input → decode → filter → encode → output 的数据流。</p>

<p data-nodeid="77">Codec 支持的编码格式常见的有 plain、json、json_lines 等，下面依次介绍。</p>
<h4 data-nodeid="234857" class="">1. Codec 插件之 plain</h4>

<p data-nodeid="79">plain 是一个空的解析器，它可以让用户自己指定格式，也就是说输入是什么格式，输出就是什么格式。下面是一个包含 plain 编码的事件配置文件：</p>
<pre class="lang-java" data-nodeid="242052"><code data-language="java">input{
    stdin {
    }
}
output{
    stdout {
         codec =&gt; <span class="hljs-string">"plain"</span>
        }
}
</code></pre>






<p data-nodeid="81">在启动 Logstash 进程后，如果输入 “hello world”，那么则输出如下结果：</p>
<pre class="lang-java" data-nodeid="246848"><code data-language="java"><span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">16</span>T07:<span class="hljs-number">16</span>:<span class="hljs-number">21.114</span>Z  nnmaster.cloud hello world
</code></pre>




<p data-nodeid="83">可以看出，输入的内容都被原样输出了，并且前面加上了时间和主机名字段。</p>
<h4 data-nodeid="248047" class="">2. Codec 插件之 json</h4>

<p data-nodeid="85">如果发送给 Logstash 的数据内容为 json 格式，则可以在 input 字段加入 codec=&gt;json 来进行解析，这样就可以根据具体内容生成字段了，方便分析和储存。如果想让 Logstash 输出为 json 格式，则可以在 output 字段中加入 codec=&gt;json。下面是一个包含 json 编码的事件配置文件：</p>
<pre class="lang-js" data-nodeid="257320"><code data-language="js">input {
    stdin {
        }
    }
output {
    stdout {
        <span class="hljs-function"><span class="hljs-params">codec</span> =&gt;</span> json
        }
}
</code></pre>








<p data-nodeid="87">同理，在启动 Logstash 进程后，如果输入“hello world”，那么输出信息为：</p>
<pre class="lang-java" data-nodeid="261956"><code data-language="java">{<span class="hljs-string">"@version"</span>:<span class="hljs-string">"1"</span>,<span class="hljs-string">"message"</span>:<span class="hljs-string">"hello world"</span>,<span class="hljs-string">"@timestamp"</span>:<span class="hljs-string">"2020-06-16T07:18:39.520Z"</span>,<span class="hljs-string">"host"</span>:<span class="hljs-string">"nnmaster.cloud"</span>}
</code></pre>




<p data-nodeid="89">这就是 json 格式的输出，可以看出，json 每个字段是 key:values 格式，多个字段之间通过逗号分隔。</p>
<h3 data-nodeid="263115" class="">Logstash 过滤器插件（Filter）</h3>

<p data-nodeid="91">丰富的过滤器插件是 Logstash 功能强大的重要因素。名为过滤器，其实它提供的不单单是过滤器的功能，还可以对进入过滤器的原始数据进行复杂的逻辑处理，甚至添加独特的事件到后续流程中。</p>
<h4 data-nodeid="264241" class="">1. Grok 正则捕获</h4>

<p data-nodeid="93">Grok 是一个十分强大的 Logstash Filter 插件，它可以通过正则解析任意文本，将非结构化日志数据格式转换为结构化的、方便查询的结构。它是目前 Logstash 中解析非结构化日志数据最好的方式。</p>
<p data-nodeid="94">Grok 的语法规则是：</p>
<pre class="lang-shell" data-nodeid="95"><code data-language="shell"><span class="hljs-meta">%</span><span class="bash">{语法: 语义}</span>
</code></pre>
<p data-nodeid="96">这里的“语法”指的是匹配模式，例如，使用 NUMBER 模式可以匹配出数字，IP 模式则会匹配出 127.0.0.1 这样的 IP 地址。比如按以下格式输入内容：</p>
<pre class="lang-java" data-nodeid="268678"><code data-language="java"><span class="hljs-number">172.16</span>.<span class="hljs-number">213.132</span> [<span class="hljs-number">16</span>/Jun/<span class="hljs-number">2020</span>:<span class="hljs-number">16</span>:<span class="hljs-number">24</span>:<span class="hljs-number">19</span> +<span class="hljs-number">0800</span>] <span class="hljs-string">"GET / HTTP/1.1"</span> <span class="hljs-number">403</span> <span class="hljs-number">5039</span>
</code></pre>




<p data-nodeid="98">那么，</p>
<ul data-nodeid="99">
<li data-nodeid="100">
<p data-nodeid="101">%{IP:clientip} 匹配模式将获得的结果为：clientip: 172.16.213.132</p>
</li>
<li data-nodeid="102">
<p data-nodeid="103">%{HTTPDATE:timestamp} 匹配模式将获得的结果为：timestamp: 16/Jun/2020:16:24:19 +0800</p>
</li>
<li data-nodeid="104">
<p data-nodeid="105">%{QS:referrer} 匹配模式将获得的结果为：referrer: "GET / HTTP/1.1"</p>
</li>
</ul>
<p data-nodeid="106">到这里为止，我们已经获取了上面输入中前三个部分的内容，分别是 clientip、timestamp 和 referrer 三个字段。如果要获取剩余部分的信息，方法类似。</p>
<p data-nodeid="107">要在线调试 Grok，<a href="http://grokdebug.herokuapp.com/" data-nodeid="565">可点击这里进行在线调试</a>，非常方便。</p>
<p data-nodeid="108">下面是一个组合匹配模式，它可以获取上面输入的所有内容：</p>
<pre class="lang-shell" data-nodeid="300839"><code data-language="shell"><span class="hljs-meta">%</span><span class="bash">{IP:clientip}\ \[%{HTTPDATE:timestamp}\]\ %{QS:referrer}\ %{NUMBER:response}\ %{NUMBER:bytes}</span>
</code></pre>





























<p data-nodeid="301948">正则匹配是非常严格的匹配，在这个组合匹配模式中，使用了转义字符 \，这是因为输入的内容中有空格和中括号。</p>
<p data-nodeid="301949">通过上面这个组合匹配模式，我们将输入的内容分成了 5 个部分，即 5 个字段。将输入内容分割为不同的数据字段，这对于日后解析和查询日志数据非常有用，这正是我们使用 grok 的目的。</p>

<p data-nodeid="111">Logstash 默认提供了近 200 个匹配模式（其实就是定义好的正则表达式）让我们来使用，可以在 Logstash 安装目录下找到。例如，我这里的路径为：</p>
<pre class="lang-java" data-nodeid="306389"><code data-language="java">/usr/local/logstash/vendor/bundle/jruby/<span class="hljs-number">2.5</span>.<span class="hljs-number">0</span>/gems/logstash-patterns-core-<span class="hljs-number">4.1</span>.<span class="hljs-number">2</span>/patterns
</code></pre>




<p data-nodeid="113">此目录下有定义好的各种匹配模式，基本匹配定义在 grok-patterns 文件中。从这些定义好的匹配模式中，可以查到上面使用的四个匹配模式对应的定义规则，如下表所示：</p>
<table data-nodeid="307499">
<thead data-nodeid="307500">
<tr data-nodeid="307501">
<th align="center" data-org-content="**匹配模式**" data-nodeid="307503"><strong data-nodeid="307523">匹配模式</strong></th>
<th align="center" data-org-content="**正则定义规则**" data-nodeid="307504"><strong data-nodeid="307527">正则定义规则</strong></th>
</tr>
</thead>
<tbody data-nodeid="307507">
<tr data-nodeid="307508">
<td align="center" data-org-content="NUMBER" data-nodeid="307509">NUMBER</td>
<td align="center" data-org-content="(?:%{BASE10NUM})" data-nodeid="307510">(?:%{BASE10NUM})</td>
</tr>
<tr data-nodeid="307511">
<td align="center" data-org-content="HTTPDATE" data-nodeid="307512">HTTPDATE</td>
<td align="center" data-org-content="%{MONTHDAY}/%{MONTH}/%{YEAR}:%{TIME} %{INT}" data-nodeid="307513">%{MONTHDAY}/%{MONTH}/%{YEAR}:%{TIME} %{INT}</td>
</tr>
<tr data-nodeid="307514">
<td align="center" data-org-content="IP" data-nodeid="307515">IP</td>
<td align="center" data-org-content="(?:%{IPV6}|%{IPV4})" data-nodeid="307516">(?:%{IPV6}|%{IPV4})</td>
</tr>
<tr data-nodeid="307517">
<td align="center" data-org-content="QS" data-nodeid="307518">QS</td>
<td align="center" data-org-content="%{QUOTEDSTRING}" data-nodeid="307519">%{QUOTEDSTRING}</td>
</tr>
</tbody>
</table>

<p data-nodeid="137">除此之外，还有很多默认定义好的匹配模式文件，比如 httpd、java、linux-syslog、redis、mongodb、nagios 等，这些已经定义好的匹配模式，可以直接在 Grok 过滤器中进行引用。当然也可以定义自己需要的匹配模式。</p>
<p data-nodeid="138">在了解完 Grok 的匹配规则之后，下面通过一个配置实例深入介绍下 Logstash 是如何将非结构化日志数据转换成结构化数据的。首先看下面的一个事件配置文件：</p>
<pre class="lang-java" data-nodeid="316375"><code data-language="java">input{
stdin{}
}
filter{
grok{
match =&gt; [<span class="hljs-string">"message"</span>,<span class="hljs-string">"%{IP:clientip}\ \[%{HTTPDATE:timestamp}\]\ %{QS:referrer}\ %{NUMBER:response}\ %{NUMBER:bytes}"</span>]
}
}
output{
stdout{
codec =&gt; <span class="hljs-string">"rubydebug"</span>
}
}
</code></pre>








<p data-nodeid="140">在这个配置文件中，输入配置成了 stdin，在 filter 中添加了 grok 过滤插件，并通过 match 来执行正则表达式解析，中括号中的正则表达式就是上面提到的组合匹配模式，然后通过 rubydebug 编码格式输出信息。这样的组合有助于调试和分析输出结果。通过此配置启动 Logstash进程后，我们仍然输入之前给出的那段内容：</p>
<pre class="lang-java" data-nodeid="320795"><code data-language="java"><span class="hljs-number">172.16</span>.<span class="hljs-number">213.132</span> [<span class="hljs-number">16</span>/Jun/<span class="hljs-number">2020</span>:<span class="hljs-number">16</span>:<span class="hljs-number">24</span>:<span class="hljs-number">19</span> +<span class="hljs-number">0800</span>] <span class="hljs-string">"GET / HTTP/1.1"</span> <span class="hljs-number">403</span> <span class="hljs-number">5039</span>
</code></pre>




<p data-nodeid="142">然后，查看 rubydebug 格式的日志输出，内容如下：</p>
<pre class="lang-java" data-nodeid="329635"><code data-language="java">{
     <span class="hljs-string">"timestamp"</span> =&gt; <span class="hljs-string">"16/Jun/2020:16:24:19 +0800"</span>,
      <span class="hljs-string">"response"</span> =&gt; <span class="hljs-string">"403"</span>,
         <span class="hljs-string">"bytes"</span> =&gt; <span class="hljs-string">"5039"</span>,
      <span class="hljs-string">"@version"</span> =&gt; <span class="hljs-string">"1"</span>,
      <span class="hljs-string">"clientip"</span> =&gt; <span class="hljs-string">"172.16.213.132"</span>,
          <span class="hljs-string">"host"</span> =&gt; <span class="hljs-string">"nnmaster.cloud"</span>,
      <span class="hljs-string">"referrer"</span> =&gt; <span class="hljs-string">"\"GET / HTTP/1.1\""</span>,
       <span class="hljs-string">"message"</span> =&gt; <span class="hljs-string">"172.16.213.132 [16/Jun/2020:16:24:19 +0800] \"GET / HTTP/1.1\" 403 5039"</span>,
    <span class="hljs-string">"@timestamp"</span> =&gt; <span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">16</span>T07:<span class="hljs-number">46</span>:<span class="hljs-number">53.120</span>Z
}
</code></pre>








<p data-nodeid="330740">从这个输出可知，通过 Grok 定义好的 5 个字段都获取到了内容，并正常输出了，看似完美，其实还有不少瑕疵。</p>
<p data-nodeid="330741">首先，message 字段也输出了完整的输入内容。这样看来，数据实质上就相当于是重复存储了两份，此时可以用 remove_field 参数来删除掉 message 字段，只保留最重要的部分。</p>

<p data-nodeid="145">其次，timestamp 字段表示日志的产生时间，而 @timestamp 默认情况下显示的是当前时间，在上面的输出中可以看出，这两个字段的时间并不一致，那么问题来了，在 ELK 日志处理系统中，@timestamp 字段会被 elasticsearch 用到，用来标注日志的生成时间。如此一来，日志生成的时间就会发生混乱，要解决这个问题，需要用到另一个插件，即 Data 插件，这个时间插件用来转换日志记录中的时间字符串，变成 LogStash::Timestamp 对象，然后转存到 @timestamp 字段里。</p>
<p data-nodeid="146">使用 Data 插件很简单，添加下面一段配置即可：</p>
<pre class="lang-java" data-nodeid="335165"><code data-language="java">date {
match =&gt; [<span class="hljs-string">"timestamp"</span>, <span class="hljs-string">"dd/MMM/yyyy:HH:mm:ss Z"</span>]
}
</code></pre>




<p data-nodeid="148">注意：时区偏移量需要用一个字母 Z 来转换。<br>
最后，将 timestamp 获得的值传给 @timestamp 后，timestamp 其实也就没有存在的意义了，所以还需要删除这个字段。</p>
<p data-nodeid="149">将上面 3 个步骤的操作统一合并到配置文件中，修改后的配置文件内容如下：</p>
<pre class="lang-java" data-nodeid="339585"><code data-language="java">input {
    stdin {}
}
filter {
    grok {
        match =&gt; { <span class="hljs-string">"message"</span> =&gt; <span class="hljs-string">"%{IP:clientip}\ \[%{HTTPDATE:timestamp}\]\ %{QS:referrer}\ %{NUMBER:response}\ %{NUMBER:bytes}"</span> }
        remove_field =&gt; [ <span class="hljs-string">"message"</span> ]
   }
date {
        match =&gt; [<span class="hljs-string">"timestamp"</span>, <span class="hljs-string">"dd/MMM/yyyy:HH:mm:ss Z"</span>]
    }
mutate {
            remove_field =&gt; [<span class="hljs-string">"timestamp"</span>]
        }
}
output {
    stdout {
        codec =&gt; <span class="hljs-string">"rubydebug"</span>
    }
}
</code></pre>




<p data-nodeid="340690">在这个配置文件中，使用了 Date 插件、mutate 插件及 remove_field 配置项，关于这两个插件后面会马上介绍。</p>
<p data-nodeid="340691">将修改后的配置文件重新运行后，仍然输入之前的那段内容：</p>

<pre class="lang-java" data-nodeid="345115"><code data-language="java"><span class="hljs-number">172.16</span>.<span class="hljs-number">213.132</span> [<span class="hljs-number">16</span>/Jun/<span class="hljs-number">2020</span>:<span class="hljs-number">16</span>:<span class="hljs-number">24</span>:<span class="hljs-number">19</span> +<span class="hljs-number">0800</span>] <span class="hljs-string">"GET / HTTP/1.1"</span> <span class="hljs-number">403</span> <span class="hljs-number">5039</span>
</code></pre>




<p data-nodeid="153">结果如下：</p>
<pre class="lang-java" data-nodeid="349535"><code data-language="java">{
      <span class="hljs-string">"@version"</span> =&gt; <span class="hljs-string">"1"</span>,
      <span class="hljs-string">"host"</span> =&gt; <span class="hljs-string">"nnmaster.cloud"</span>,
      <span class="hljs-string">"bytes"</span> =&gt; <span class="hljs-string">"5039"</span>,
      <span class="hljs-string">"@timestamp"</span> =&gt; <span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">16</span>T08:<span class="hljs-number">24</span>:<span class="hljs-number">19.000</span>Z,
      <span class="hljs-string">"referrer"</span> =&gt; <span class="hljs-string">"\"GET / HTTP/1.1\""</span>,
      <span class="hljs-string">"response"</span> =&gt; <span class="hljs-string">"403"</span>,
      <span class="hljs-string">"clientip"</span> =&gt; <span class="hljs-string">"172.16.213.132"</span>
}
</code></pre>




<p data-nodeid="155">这就是我们需要的最终结果。</p>
<h4 data-nodeid="350640" class="">2. 时间处理 (Date)</h4>

<p data-nodeid="157">Date 插件对于排序事件和回填旧数据尤其重要，它可以用来转换日志记录中的时间字段，变成 LogStash::Timestamp 对象，然后转存到 @timestamp 字段里，在上一课时中已经做过简单的介绍。</p>
<p data-nodeid="158">下面是 Date 插件的一个配置示例（这里仅仅列出 filter 部分）：</p>
<pre class="lang-java" data-nodeid="354949"><code data-language="java">filter {
    grok {
        match =&gt; [<span class="hljs-string">"message"</span>, <span class="hljs-string">"%{HTTPDATE:timestamp}"</span>]
    }
    date {
        match =&gt; [<span class="hljs-string">"timestamp"</span>, <span class="hljs-string">"dd/MMM/yyyy:HH:mm:ss Z"</span>]
    }
}
</code></pre>




<p data-nodeid="160">为什么要使用这个 Date 插件呢？主要有两方面原因。</p>
<p data-nodeid="161">一方面由于 Logstash 会给收集到的每条日志自动打上时间戳（即 @timestamp），但是这个时间戳记录的是 input 接收数据的时间，而不是日志生成的时间（因为日志生成时间与 input 接收的时间肯定不同），这样就可能导致搜索数据时产生混乱。</p>
<p data-nodeid="162">另一方面，不知道大家是否注意到了，在上面那段 rubydebug 编码格式的输出中，@timestamp 字段虽然已经获取了 timestamp 字段的时间，但是仍然比北京时间早了 8 个小时，这是因为在 Elasticsearch 内部，对时间类型字段都是统一采用 UTC 时间，而日志统一采用 UTC 时间存储，是国际安全、运维界的一个共识。其实这并不影响什么，因为 ELK 已经给出了解决方案，那就是在 Kibana 平台上，程序会自动读取浏览器的当前时区，然后在 Web 页面自动将 UTC 时间转换为当前时区的时间。</p>
<h4 data-nodeid="356026" class="">3. 数据修改（Mutate）</h4>

<p data-nodeid="164">Mutate 插件是 Logstash 另一个非常重要插件，它提供了丰富的基础类型数据处理能力，包括重命名、删除、替换和修改日志事件中的字段。这里重点介绍下 Mutate 插件的字段类型转换功能（convert）、正则表达式替换匹配字段功能（gsub）、分隔符分割字符串为数组功能（split）、重命名字段功能（rename）、删除字段功能（remove_field）的具体实现方法。</p>
<p data-nodeid="165">（1）字段类型转换功能</p>
<p data-nodeid="166">Mutate 可以设置的转换类型有 integer、float 和 string。下面是一个关于 mutate 字段类型转换的示例（仅列出 filter 部分）：</p>
<pre class="lang-java" data-nodeid="360199"><code data-language="java">filter {
    mutate {
        convert =&gt; [<span class="hljs-string">"filed_name"</span>, <span class="hljs-string">"integer"</span>]
    }
}
</code></pre>




<p data-nodeid="361242">这个示例表示将 filed_name 字段类型修改为 integer。</p>
<p data-nodeid="361243" class="">（2）正则表达式替换匹配字段</p>

<p data-nodeid="169">gsub 可以通过正则表达式替换字段中匹配到的值，只对字符串字段有效。下面是一个关于 mutate 插件中 gsub 的示例（仅列出 filter 部分）：</p>
<pre class="lang-java" data-nodeid="365419"><code data-language="java">filter {
    mutate {
        gsub =&gt; [<span class="hljs-string">"filed_name_1"</span>, <span class="hljs-string">"/"</span> , <span class="hljs-string">"_"</span>]
    }
}
</code></pre>




<p data-nodeid="366462">这个示例表示将 filed_name1 字段中所有 "/" 字符替换为 "_"。</p>
<p data-nodeid="366463">（3）分隔符分割字符串为数组</p>

<p data-nodeid="172">split 可以通过指定的分隔符分割字段中的字符串为数组。下面是一个关于 mutate 插件中 split 的示例（仅列出 filter 部分）：</p>
<pre class="lang-java" data-nodeid="370647"><code data-language="java">filter {
    mutate {
        split =&gt; {<span class="hljs-string">"filed_name_2"</span>, <span class="hljs-string">"|"</span>}
    }
}
</code></pre>




<p data-nodeid="371690">这个示例表示将 filed_name_2 字段以 "|" 为区间分隔成数组形式。</p>
<p data-nodeid="371691">（4）重命名字段</p>

<p data-nodeid="175">rename 可以实现重命名某个字段的功能。下面是一个关于 mutate 插件中 rename 的示例（仅列出 filter 部分）：</p>
<pre class="lang-java" data-nodeid="375873"><code data-language="java">filter {
    mutate {
        rename =&gt; {<span class="hljs-string">"old_field"</span> =&gt; <span class="hljs-string">"new_field"</span>}
    }
}
</code></pre>




<p data-nodeid="376916">这个示例表示将字段 old_field 重命名为 new_field。</p>
<p data-nodeid="376917">（5）删除字段</p>

<p data-nodeid="178">remove_field 可以实现删除某个字段的功能。下面是一个关于 mutate 插件中 remove_field 的示例（仅列出 filter 部分）：</p>
<pre class="lang-java" data-nodeid="381095"><code data-language="java">filter {
    mutate {
        remove_field  =&gt;  [<span class="hljs-string">"timestamp"</span>]
    }
}
</code></pre>




<p data-nodeid="180">这个示例表示将字段 timestamp 删除。 在本课时的最后，我们将上面 mutate 插件的几个功能点整合到一个完整的配置文件中，以验证 mutate 插件实现的功能细节，配置文件内容如下：</p>
<pre class="lang-java" data-nodeid="385267"><code data-language="java">input {
    stdin {}
}
filter {
    grok {
        match =&gt; { <span class="hljs-string">"message"</span> =&gt; <span class="hljs-string">"%{IP:clientip}\ \[%{HTTPDATE:timestamp}\]\ %{QS:referrer}\ %{NUMBER:response}\ %{NUMBER:bytes}"</span> }
        remove_field =&gt; [ <span class="hljs-string">"message"</span> ]
   }
date {
        match =&gt; [<span class="hljs-string">"timestamp"</span>, <span class="hljs-string">"dd/MMM/yyyy:HH:mm:ss Z"</span>]
    }
mutate {
           rename =&gt; { <span class="hljs-string">"response"</span> =&gt; <span class="hljs-string">"response_new"</span> }
           convert =&gt; [ <span class="hljs-string">"response"</span>,<span class="hljs-string">"float"</span> ]
           gsub =&gt; [<span class="hljs-string">"referrer"</span>,<span class="hljs-string">"\""</span>,<span class="hljs-string">""</span>]
           remove_field =&gt; [<span class="hljs-string">"timestamp"</span>]
           split =&gt; [<span class="hljs-string">"clientip"</span>, <span class="hljs-string">"."</span>]
        }
}
output {
    stdout {
        codec =&gt; <span class="hljs-string">"rubydebug"</span>
    }
}
</code></pre>




<p data-nodeid="182">将此配置文件运行后，仍然输入之前的那段内容：</p>
<pre class="lang-java" data-nodeid="389439"><code data-language="java"><span class="hljs-number">172.16</span>.<span class="hljs-number">213.132</span> [<span class="hljs-number">16</span>/Jun/<span class="hljs-number">2020</span>:<span class="hljs-number">16</span>:<span class="hljs-number">24</span>:<span class="hljs-number">19</span> +<span class="hljs-number">0800</span>] <span class="hljs-string">"GET / HTTP/1.1"</span> <span class="hljs-number">403</span> <span class="hljs-number">5039</span>
</code></pre>




<p data-nodeid="184">输出结果如下：</p>
<pre class="lang-java" data-nodeid="393611"><code data-language="java">{
        <span class="hljs-string">"host"</span> =&gt; <span class="hljs-string">"nnmaster.cloud"</span>,
        <span class="hljs-string">"response_new"</span> =&gt; <span class="hljs-string">"403"</span>,
        <span class="hljs-string">"clientip"</span> =&gt; [
        [<span class="hljs-number">0</span>] <span class="hljs-string">"172"</span>,
        [<span class="hljs-number">1</span>] <span class="hljs-string">"16"</span>,
        [<span class="hljs-number">2</span>] <span class="hljs-string">"213"</span>,
        [<span class="hljs-number">3</span>] <span class="hljs-string">"132"</span>
    ],
           <span class="hljs-string">"bytes"</span> =&gt; <span class="hljs-string">"5039"</span>,
      <span class="hljs-string">"@timestamp"</span> =&gt; <span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">16</span>T08:<span class="hljs-number">24</span>:<span class="hljs-number">19.000</span>Z,
        <span class="hljs-string">"referrer"</span> =&gt; <span class="hljs-string">"GET / HTTP/1.1"</span>,
        <span class="hljs-string">"@version"</span> =&gt; <span class="hljs-string">"1"</span>
}
</code></pre>




<p data-nodeid="186">从这个输出中，可以很清楚地看到，mutate 插件是如何操作日志事件中字段的。</p>
<h3 data-nodeid="394654" class="">Logstash 输出插件（Output）</h3>

<p data-nodeid="188">Output 是 Logstash 的最后阶段，一个事件可以经过多个输出，而一旦所有输出处理完成后，整个事件就执行完成。 一些常用的输出包括：</p>
<ul data-nodeid="189">
<li data-nodeid="190">
<p data-nodeid="191">file，表示将日志数据写入磁盘上的文件；</p>
</li>
<li data-nodeid="192">
<p data-nodeid="193">elasticsearch，表示将日志数据发送给 Elasticsearch，它可以高效方便和易于查询的保存数据。</p>
</li>
</ul>
<p data-nodeid="194">此外，Logstash 还支持输出到 Nagios、HDFS、Email（发送邮件）和 Exec（调用命令执行）。</p>
<h4 data-nodeid="395670" class="">1. 输出到标准输出（stdout）</h4>

<p data-nodeid="196">stdout 与之前介绍过的 stdin 插件一样，它是最基础和简单的输出插件。下面是一个配置实例：</p>
<pre class="lang-js" data-nodeid="403535"><code data-language="js">output {
    stdout {
        <span class="hljs-function"><span class="hljs-params">codec</span> =&gt;</span> rubydebug
    }
}
</code></pre>








<p data-nodeid="198">stdout 插件主要的功能和用途是用于调试，该插件在前面已经多次使用过，这里就不再过多介绍了。</p>
<h4 data-nodeid="404518" class="">2. 保存为文件（file）</h4>

<p data-nodeid="200">file 插件可以将输出保存到一个文件中，配置实例如下：</p>
<pre class="lang-java" data-nodeid="408339"><code data-language="java">output {
    file {
        path =&gt; <span class="hljs-string">"/data/log/%{+yyyy-MM-dd}/%{host}_%{+HH}.log"</span>
    }
</code></pre>




<p data-nodeid="202">在上面这个配置中，使用了变量匹配，用于自动匹配时间和主机名，这在实际使用中很有帮助。</p>
<p data-nodeid="203">file 插件默认会以 JSON 形式将数据保存到指定的文件中，而如果只希望按照日志的原始格式保存的话，就需要通过 codec 编码方式自定义 %{message}，将日志按照原始格式保存输出到文件。配置实例如下：</p>
<pre class="lang-java" data-nodeid="412159"><code data-language="java">output {
    file {
        path =&gt; <span class="hljs-string">"/data/log/%{+yyyy-MM-dd}/%{host}_%{+HH}.log.gz"</span>
        codec =&gt; line { format =&gt; <span class="hljs-string">"%{message}"</span>}
        gzip =&gt; <span class="hljs-keyword">true</span>
    }
</code></pre>




<p data-nodeid="205">在这个配置中，使用了 codec 编码方式，将输出日志转换为原始格式，同时，输出数据文件还开启了 gzip 压缩，自动将输出保存为压缩文件格式。</p>
<h3 data-nodeid="206">总结</h3>
<p data-nodeid="413114">本课时注意讲解了 Logstash 的配置文件编写方法，以及输入插件、编码插件、过滤插件和输出插件的使用方法，这些插件的熟练掌握对于 Logstash 来说至关重要。因为 Logstash 所有功能的实现都是建立在插件基础上的。Logstash 默认自带的插件已经能够满足我们 80% 左右的应用需求。</p>

---

### 精选评论


