<p data-nodeid="4995" class="">上一课时之前，我们的接口都是在代码中模拟假数据，并没有从服务端获取数据，但是在实际开发中，必须与服务端进行交互。本课时主要介绍在 Flutter 中常见的网络传输协议序列化方式，并对其中比较常用的协议进行简单实践，最后再通过 JSON 协议来完善本课时的 api 部分的代码。</p>
<h3 data-nodeid="4996">常见的 APP 网络传输协议序列化方式</h3>
<p data-nodeid="4997">常见的传输协议有三种：XML 、JSON 和 Protocol Buffer。我们先来对比下这三种协议，我会分别从 Flutter 中的实现、序列化后的数据长度、Flutter 中反序列化性能三个方面来讲解。我先将本课时中的一段基础的数据格式用来做效果演示，测试数据如下：</p>
<pre class="lang-coffeescript" data-nodeid="4998"><code data-language="coffeescript">nickName = <span class="hljs-string">'test-pb'</span>; 
uid = <span class="hljs-string">'3001'</span>; 
headerUrl = <span class="hljs-string">'http://image.biaobaiju.com/uploads/20180211/00/1518279967-IAnVyPiRLK.jpg'</span>;
</code></pre>
<p data-nodeid="4999">上面的是用户信息接口，接下来我们使用这三种方式来实现这个接口。</p>
<h4 data-nodeid="5000">XML</h4>
<p data-nodeid="5001">XML 指可扩展标记语言（eXtensible Markup Language）是一种通用的重量级数据交换格式，以文本结构存储。</p>
<p data-nodeid="5002">在 Flutter 中有一个解析 XML 的第三方库 <a href="https://pub.dev/packages/xml2json" data-nodeid="5095">xml2json</a>，将服务端的 XML 解析为 JSON 格式，因为是第三方库，因此需要在 pubspec.yaml 中增加该库的依赖，然后更新本地库。接下来我们实现具体的代码，在 lib 目录下新建 api_xml ，然后在目录下创建 api_xml/user_info/index.dart 。创建完成后，我们来实现 user_info/index.dart 的逻辑。</p>
<p data-nodeid="5003">首先需要增加第三方库的引用。</p>
<pre class="lang-dart" data-nodeid="5004"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'dart:convert'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/user_info.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:xml2json/xml2json.dart'</span>;
</code></pre>
<p data-nodeid="5005">接下来实现 ApiXmlUserInfoIndex 类中的 getSelfUserInfo 方法，后续 getSelfUserInfo 会是一个异步网络请求方法，因此将返回类型修改为 Future<code data-backticks="1" data-nodeid="5107">&lt;StructUserInfo&gt;</code>，具体实现逻辑如下：</p>
<pre class="lang-dart" data-nodeid="5006"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取自己的个人信息 </span></span>
<span class="hljs-keyword">static</span> Future&lt;StructUserInfo&gt; getSelfUserInfo() <span class="hljs-keyword">async</span>{ 
  <span class="hljs-comment">// 模拟xml假数据 </span>
  <span class="hljs-keyword">final</span> userInfoXml = <span class="hljs-string">'''&lt;?xml version="1.0"?&gt; 
  &lt;userInfo&gt; 
    &lt;nickName&gt;test&lt;/nickName&gt; 
    &lt;uid&gt;3001&lt;/uid&gt; 
    &lt;headerUrl&gt;http://image.biaobaiju.com/uploads/20180211/00/1518279967-IAnVyPiRLK.jpg&lt;/headerUrl&gt; 
  &lt;/userInfo&gt;'''</span>; 

  <span class="hljs-comment">// 记录当前时间 </span>
  <span class="hljs-built_in">int</span> currentTime = <span class="hljs-keyword">new</span> <span class="hljs-built_in">DateTime</span>.now().microsecondsSinceEpoch; 
  Xml2Json xml2json = Xml2Json(); 
  xml2json.parse(userInfoXml); 
  <span class="hljs-comment">// 转化xml数据 </span>
  <span class="hljs-keyword">final</span> userInfoStr = xml2json.toGData(); 
  <span class="hljs-built_in">print</span>(<span class="hljs-string">'xml length'</span>); 
  <span class="hljs-built_in">print</span>(userInfoStr.length); 

  <span class="hljs-built_in">int</span> jsonStartTime = <span class="hljs-keyword">new</span> <span class="hljs-built_in">DateTime</span>.now().microsecondsSinceEpoch; 
  <span class="hljs-keyword">final</span> userInfo = json.decode(userInfoStr); 
  <span class="hljs-comment">// 打印解析json时间 </span>
  <span class="hljs-built_in">print</span>(<span class="hljs-string">'json decode time'</span>); 
  <span class="hljs-built_in">print</span>(<span class="hljs-keyword">new</span> <span class="hljs-built_in">DateTime</span>.now().microsecondsSinceEpoch - jsonStartTime); 

  <span class="hljs-comment">// 打印整体解析时间 </span>
  <span class="hljs-built_in">print</span>(<span class="hljs-string">'xml decode time'</span>); 
  <span class="hljs-built_in">print</span>(<span class="hljs-keyword">new</span> <span class="hljs-built_in">DateTime</span>.now().microsecondsSinceEpoch - currentTime); 
  <span class="hljs-keyword">return</span> StructUserInfo( 
      userInfo[<span class="hljs-string">'userInfo'</span>][<span class="hljs-string">'uid'</span>][<span class="hljs-string">'\$t'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">String</span>, 
      userInfo[<span class="hljs-string">'userInfo'</span>][<span class="hljs-string">'nickName'</span>][<span class="hljs-string">'\$t'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">String</span>, 
      userInfo[<span class="hljs-string">'userInfo'</span>][<span class="hljs-string">'headerUrl'</span>][<span class="hljs-string">'\$t'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">String</span> 
  ); 
}
</code></pre>
<p data-nodeid="5007">上述代码首先在第 4 行模拟一个 XML 数据，在第 12 行记录开始解析时间，第 28 行打印整体 XML 解析时间，在第 24 行打印 JSON 的解析时间。XML 的解析过程是先将 XML 转化为一个 JSON 字符串，然后再通过 convert 转化为 JSON。在 main.dart 中引入该文件，并调用 getSelfUserInfo 方法，可以看到如下的打印信息。</p>
<pre class="lang-plain" data-nodeid="5008"><code data-language="plain">flutter: xml length 
flutter: 180 
flutter: json decode time 
flutter: 200 
flutter: xml decode time 
flutter: 2000
</code></pre>
<p data-nodeid="5009">从解析过程来看，XML 的解析性能肯定是比较差的，因为最终还是需要将 XML 转化为 JSON 来处理，接下来我们看下 JSON 的解析实现方式。</p>
<h4 data-nodeid="5010">JSON</h4>
<p data-nodeid="5011">JSON（JavaScript Object Notation）是一种轻量级的数据交换格式。 易于人阅读和编写，同时也易于机器解析和生成。 它是基于 JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999 的一个子集。</p>
<p data-nodeid="5012">在 Flutter 中，JSON 解析有专门的 dart 原生库支持——dart:convert。同样我们去实现 XML 例子中的 user_info/index.dart，我们以 api/user_info/index.dart 为例子来实现，在原来代码基础上，我们增加打印解析时间和 JSON 长度，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="5013"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取自己的个人信息 </span></span>
  <span class="hljs-keyword">static</span> Future&lt;StructUserInfo&gt; getSelfUserInfo() <span class="hljs-keyword">async</span>{ 
    <span class="hljs-built_in">String</span> jsonStr = <span class="hljs-string">'{"nickName":"test","uid":"3001","headerUrl":"http://image.biaobaiju.com/uploads/20180211/00/1518279967-IAnVyPiRLK.jpg"}'</span>; 
    <span class="hljs-built_in">print</span>(<span class="hljs-string">'json length'</span>); 
    <span class="hljs-built_in">print</span>(jsonStr.length); 
    <span class="hljs-built_in">int</span> currentTime = <span class="hljs-keyword">new</span> <span class="hljs-built_in">DateTime</span>.now().microsecondsSinceEpoch; 
    <span class="hljs-keyword">final</span> jsonInfo = json.decode(jsonStr) <span class="hljs-keyword">as</span> <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt;; 
    <span class="hljs-built_in">print</span>(<span class="hljs-string">'json parse time'</span>); 
    <span class="hljs-built_in">print</span>(<span class="hljs-keyword">new</span> <span class="hljs-built_in">DateTime</span>.now().microsecondsSinceEpoch - currentTime); 
    <span class="hljs-keyword">return</span> StructUserInfo.fromJson(jsonInfo); 
  }
</code></pre>
<p data-nodeid="5014">上面代码较 XML 简单一些，第 3 行创建假数据，然后在第 7 行进行解析。在代码第 5 行，打印 JSON 长度，第 9 行打印具体的解析时间，在 main.dart 执行该函数，可以看到如下打印数据。</p>
<pre class="lang-java" data-nodeid="5015"><code data-language="java">flutter: json length 
flutter: <span class="hljs-number">119</span> 
flutter: json parse time 
flutter: <span class="hljs-number">420</span>
</code></pre>
<p data-nodeid="5016">与 XML 对比，从解析时间和传递数据长度来看，都是较优的，接下来我们看下 Protocol Buffer 的实现、相关解析时长和具体的数据长度。</p>
<h4 data-nodeid="5017">Protocol Buffer</h4>
<p data-nodeid="5018">Protocol Buffer 是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC 数据交换格式，可用于通信协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。</p>
<p data-nodeid="5019">在 Flutter 中应用 Protocol Buffer 需要下面几个过程。</p>
<p data-nodeid="5020">1.<strong data-nodeid="5128">安装 Protocol Buffer 工具</strong>，在 Mac 应用如下命令。</p>
<pre class="lang-js" data-nodeid="5021"><code data-language="js">brew install protobuf
</code></pre>
<p data-nodeid="5022">如果是在 Windows 或者 Linux 上，则前往（ <a href="https://github.com/protocolbuffers/protobuf/releases?after=v3.5.0" data-nodeid="5132">https://github.com/protocolbuffers/protobuf/releases?after=v3.5.0</a>）解压安装即可。</p>
<p data-nodeid="5023">2.<strong data-nodeid="5141">安装 protoc_plugin 插件</strong>，在 Mac 或者 Linux 应用如下命令安装。</p>
<pre class="lang-plain" data-nodeid="5024"><code data-language="plain">pub global activate protoc_plugin
</code></pre>
<p data-nodeid="5025">如果在 Windows 上没有这个支持，你在 Windows 上只能通过虚拟机的方式。</p>
<p data-nodeid="5026">3.<strong data-nodeid="5150">在 lib 同级目录下创建 protos</strong> 用来存放所需要的 Protocol Buffer 文件，这里我们创建了一个 user_info.proto ，然后添加下面的代码：</p>
<pre class="lang-java" data-nodeid="5027"><code data-language="java">syntax = <span class="hljs-string">"proto3"</span>; 
option java_package = <span class="hljs-string">"pro.two_you_friend"</span>; 
message UserInfoRsp { 
    string nickName = <span class="hljs-number">1</span>; 
    string headerUrl = <span class="hljs-number">2</span>; 
    string uid = <span class="hljs-number">3</span>; 
}
</code></pre>
<p data-nodeid="5028">上面的代码就是创建一个 Protocol Buffer 协议，该协议数据结构就是一个 UserInfo 的结构，具体关于 Protocol Buffer 的协议，可以<a href="https://developers.google.com/protocol-buffers/docs/proto3" data-nodeid="5154">参考官网</a>。</p>
<p data-nodeid="5029">4.创建完成 Protocol Buffer 协议后，我们再<strong data-nodeid="5161">将 Protocol Buffer 文件转化为 Dart 文件</strong>，在项目根目录，也就是 lib 同级目录，运行下面命令。</p>
<pre class="lang-shell" data-nodeid="5030"><code data-language="shell">protoc --dart_out=./lib ./protos/* --plugin=protoc-gen-dart=$HOME/.pub-cache/bin/protoc-gen-dart
</code></pre>
<p data-nodeid="5031">其中 dart_out 就是转化后的 dart 文件存放路径，会默认带上原有 protos 目录。--plugin 就是需要使用到的插件，这里的路径就是第二步安装的插件位置。</p>
<p data-nodeid="5032">5.运行成功后，<strong data-nodeid="5170">会在 lib 目录下创建 protos 目录</strong>，并生成如图 1 的目录结构；</p>
<p data-nodeid="5033"><img src="https://s0.lgstatic.com/i/image/M00/3A/39/Ciqc1F8hN7qARIvYAACMRuwnLuo133.png" alt="image (14).png" data-nodeid="5173"></p>
<div data-nodeid="5034"><p style="text-align:center">图 1 生成的 Protocol Buffer 目录结构</p></div>
<p data-nodeid="5035">生成完成以后，这时候是会提示报错的，因为在 user_info.pb.dart 中引用了 package:protobuf/protobuf.dart 这个库。接下来我们就需要去修改 pubspec.yaml ，添加 Protocol Buffer（ protobuf: ^1.0.1 ）第三方库的依赖，添加完成后更新本地库。</p>
<p data-nodeid="5036">以上就完成了整个 Protocol Buffer 的创建到转化，接下来我们看下如何在 Flutter 应用，同样和 XML 以及 JSON 一样，我们继续在 lib 目录下新建一个 api_pb 文件夹，用来存放 Protocol Buffer 相关的 API 协议，这里为了演示，只创建 api_pb/user_info/index.dart 。接下来我们看下具体的代码逻辑。</p>
<p data-nodeid="5037">先引入相应的库文件，其中第 2 行就是相应的 Protocol Buffer 文件。</p>
<pre class="lang-dart" data-nodeid="5038"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/user_info.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/protos/user_info.pb.dart'</span>;
</code></pre>
<p data-nodeid="5039">接下来我们看下 ApiPbUserInfoIndex 类中创建 Protocol Buffer 的代码部分，这部分逻辑放在 createUserInfo 函数中，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="5040"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">生成二进制内容，测试文件 </span></span>
<span class="hljs-keyword">static</span> <span class="hljs-built_in">List</span>&lt;<span class="hljs-built_in">int</span>&gt; createUserInfo() { 
  UserInfoRsp userInfoRsp = UserInfoRsp(); 
  userInfoRsp.nickName = <span class="hljs-string">'test'</span>; 
  userInfoRsp.uid = <span class="hljs-string">'3001'</span>; 
  userInfoRsp.headerUrl = <span class="hljs-string">'http://image.biaobaiju.com/uploads/20180211/00/1518279967-IAnVyPiRLK.jpg'</span>; 
  <span class="hljs-built_in">List</span>&lt;<span class="hljs-built_in">int</span>&gt; retInfo = userInfoRsp.writeToBuffer(); 
  <span class="hljs-keyword">return</span> retInfo; 
}
</code></pre>
<p data-nodeid="5041">代码的第 2 行就是创建 Protocol Buffer 中的 Message 类，也就是我们的 UserInfo 数据结构，然后根据其数据结构，设置具体的字段值，最后调用 writeToBuffer 转化为二进制数据。</p>
<p data-nodeid="5042">应用上面生成的二进制数据，我们再来实现 getSelfUserInfo 方法，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="5043"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">获取自己的个人信息 </span></span>
<span class="hljs-keyword">static</span> Future&lt;StructUserInfo&gt; getSelfUserInfo() <span class="hljs-keyword">async</span>{ 
  <span class="hljs-comment">// 该数据涞源createUserInfo函数 </span>
  <span class="hljs-built_in">int</span> currentTime = <span class="hljs-keyword">new</span> <span class="hljs-built_in">DateTime</span>.now().microsecondsSinceEpoch; 
  UserInfoRsp userInfoRsp = UserInfoRsp.fromBuffer( 
      [ 
        <span class="hljs-number">10</span>, <span class="hljs-number">4</span>, <span class="hljs-number">116</span>, <span class="hljs-number">101</span>, <span class="hljs-number">115</span>, <span class="hljs-number">116</span>, <span class="hljs-number">18</span>, <span class="hljs-number">72</span>, <span class="hljs-number">104</span>, <span class="hljs-number">116</span>, 
        <span class="hljs-number">116</span>, <span class="hljs-number">112</span>, <span class="hljs-number">58</span>, <span class="hljs-number">47</span>, <span class="hljs-number">47</span>, <span class="hljs-number">105</span>, <span class="hljs-number">109</span>, <span class="hljs-number">97</span>, <span class="hljs-number">103</span>, <span class="hljs-number">101</span>, 
        <span class="hljs-number">46</span>, <span class="hljs-number">98</span>, <span class="hljs-number">105</span>, <span class="hljs-number">97</span>, <span class="hljs-number">111</span>, <span class="hljs-number">98</span>, <span class="hljs-number">97</span>, <span class="hljs-number">105</span>, <span class="hljs-number">106</span>, <span class="hljs-number">117</span>, 
        <span class="hljs-number">46</span>, <span class="hljs-number">99</span>, <span class="hljs-number">111</span>, <span class="hljs-number">109</span>, <span class="hljs-number">47</span>, <span class="hljs-number">117</span>, <span class="hljs-number">112</span>, <span class="hljs-number">108</span>, <span class="hljs-number">111</span>, <span class="hljs-number">97</span>, 
        <span class="hljs-number">100</span>, <span class="hljs-number">115</span>, <span class="hljs-number">47</span>, <span class="hljs-number">50</span>, <span class="hljs-number">48</span>, <span class="hljs-number">49</span>, <span class="hljs-number">56</span>, <span class="hljs-number">48</span>, <span class="hljs-number">50</span>, <span class="hljs-number">49</span>, <span class="hljs-number">49</span>, 
        <span class="hljs-number">47</span>, <span class="hljs-number">48</span>, <span class="hljs-number">48</span>, <span class="hljs-number">47</span>, <span class="hljs-number">49</span>, <span class="hljs-number">53</span>, <span class="hljs-number">49</span>, <span class="hljs-number">56</span>, <span class="hljs-number">50</span>, <span class="hljs-number">55</span>, <span class="hljs-number">57</span>, 
        <span class="hljs-number">57</span>, <span class="hljs-number">54</span>, <span class="hljs-number">55</span>, <span class="hljs-number">45</span>, <span class="hljs-number">73</span>, <span class="hljs-number">65</span>, <span class="hljs-number">110</span>, <span class="hljs-number">86</span>, <span class="hljs-number">121</span>, <span class="hljs-number">80</span>, 
        <span class="hljs-number">105</span>, <span class="hljs-number">82</span>, <span class="hljs-number">76</span>, <span class="hljs-number">75</span>, <span class="hljs-number">46</span>, <span class="hljs-number">106</span>, <span class="hljs-number">112</span>, <span class="hljs-number">103</span>, <span class="hljs-number">26</span>, 
        <span class="hljs-number">4</span>, <span class="hljs-number">51</span>, <span class="hljs-number">48</span>, <span class="hljs-number">48</span>, <span class="hljs-number">49</span> 
      ] 
  ); 
  <span class="hljs-built_in">print</span>(<span class="hljs-string">'pb length'</span>); 
  <span class="hljs-built_in">print</span>(userInfoRsp.toString().length); 
  <span class="hljs-built_in">int</span> dfTime = <span class="hljs-keyword">new</span> <span class="hljs-built_in">DateTime</span>.now().microsecondsSinceEpoch - currentTime; 
  <span class="hljs-built_in">print</span>(<span class="hljs-string">'pb decode time'</span>); 
  <span class="hljs-built_in">print</span>(dfTime); 
  <span class="hljs-keyword">return</span> StructUserInfo( 
    userInfoRsp.uid, 
    userInfoRsp.nickName, 
    userInfoRsp.headerUrl 
  ); 
}
</code></pre>
<p data-nodeid="5044">代码第 5 行是应用 createUserInfo 生成的二进制数据，利用该二进制数据调用 fromBuffer 转化为 Protocol Buffer 对象，返回的对象可以直接获取到 StructUserInfo 的相应字段： uid、nickName 和 headerUrl，具体代码在第 25 到第 28行。第 18 行打印字符串长度，第 23 行打印反序列化时间。运行上面的代码，可以看到如下打印数据：</p>
<pre class="lang-java" data-nodeid="5045"><code data-language="java">flutter: pb length 
flutter: <span class="hljs-number">109</span> 
flutter: pb decode time 
flutter: <span class="hljs-number">383</span>
</code></pre>
<p data-nodeid="5046">长度和解析时长相对 JSON 协议又减少了一些，因此在带宽和解析性能方面都是优于 JSON 和 XML。由于本课时中还没有实现服务端代码，我们只能借助第三方 Mock 平台来实现网络调用，因此这里会以 JSON 协议为参考来实践本课时的 api 层代码逻辑。在实际应用中，我更倾向大家使用 Protocol Buffer 。</p>
<p data-nodeid="5047">以上就是三种协议在 Flutter 中的应用尝试和对比，基于数据长度和解析性能对比（由于跑的数据总量不够大，因此单次运行会存在样本误差），XML 是最差的，JSON 相对较好，Protocol Buffer 是最优的，不过可读性最差，具体对比看下表格 1。</p>
<p data-nodeid="5048"><img src="https://s0.lgstatic.com/i/image/M00/3A/45/CgqCHl8hN_CAHkTmAABBGfUbGT4406.png" alt="image (15).png" data-nodeid="5193"></p>
<div data-nodeid="5049"><p style="text-align:center">表格 1 整体数据对比情况</p></div>
<h3 data-nodeid="5050">代码实践</h3>
<p data-nodeid="5242" class="te-preview-highlight">介绍完常见的网络传输协议序列化方式，接下来就使用 JSON 的传输协议来完善我们 api 逻辑。这里会应用到一个第三方的 Mock 平台。主要是 Mock 以下几个接口协议，如图 2 所示的结构列表。</p>

<p data-nodeid="5052"><img src="https://s0.lgstatic.com/i/image/M00/3A/3A/Ciqc1F8hOAmAc5eCAAH87tkUVJM571.png" alt="image (16).png" data-nodeid="5198"></p>
<div data-nodeid="5053"><p style="text-align:center">图 2 Mock 协议列表</p></div>
<p data-nodeid="5054">有了具体协议 Mock 协议后，我们再来实现 Flutter 中的代码。首先我们需要创建一个通用的网络请求的类，这个类我们存放在 util/tools 目录下，命名为 call_server.dart 。Flutter 中的网络协议需要使用到 <a href="https://pub.dev/packages/dio" data-nodeid="5204">dio</a> 这个第三方库，同样还是需要在 pubspec.yaml 增加依赖，然后更新本地库文件。接下来我们看下 call_server.dart 的代码实现。</p>
<h4 data-nodeid="5055">通用网络请求类实现</h4>
<p data-nodeid="5056">该通用网络请求类，文件存放在源码中的 lib/util/tools/call_server.dart ，接下来我们看下它的实现逻辑。</p>
<p data-nodeid="5057">首先还是引入相应的库文件</p>
<pre class="lang-dart" data-nodeid="5058"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'dart:convert'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:dio/dio.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/tools/json_config.dart'</span>;
</code></pre>
<p data-nodeid="5059">第 1 个库是数据转化类的原生库，第 2 个库是网络请求库，第 3 个库是我们自己实现的一个工具库，该库的作用是读取一个 JSON 配置文件。</p>
<p data-nodeid="5060">接下来我们实现 CallServer 类，在类中新增一个 get 方法，这里需要注意因为 dio 网络请求是一个异步方法，因此这里需要将 get 设计为一个 async 的方法，并返回的是一个 Future 类型，具体代码如下：</p>
<pre class="lang-dart" data-nodeid="5061"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">统一调用API接口 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CallServer</span> </span>{ 
  <span class="hljs-comment">/// <span class="markdown">get 方法 </span></span>
  <span class="hljs-keyword">static</span> Future&lt;<span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt;&gt; <span class="hljs-keyword">get</span> 
  } 
}
</code></pre>
<p data-nodeid="5062">因为网络请求异步返回的是一个 JSON 协议，因此需要设置返回的数据结构为 Map&lt;String, dynamic&gt; 。接下来我们看下具体的函数代码逻辑。</p>
<pre class="lang-dart" data-nodeid="5063"><code data-language="dart"><span class="hljs-comment">// 根据类型，获取api具体信息 </span>
<span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; apis = <span class="hljs-keyword">await</span> JsonConfig.getConfig(<span class="hljs-string">'api'</span>); 
<span class="hljs-keyword">if</span>(apis == <span class="hljs-keyword">null</span>) { 
  <span class="hljs-keyword">return</span> {<span class="hljs-string">"ret"</span> : <span class="hljs-keyword">false</span>}; 
} 
<span class="hljs-built_in">String</span> callApi = apis[apiName][<span class="hljs-string">'apiUrl'</span>] <span class="hljs-keyword">as</span> <span class="hljs-built_in">String</span>; 
<span class="hljs-comment">// 处理异常情况 </span>
<span class="hljs-keyword">if</span>(callApi == <span class="hljs-keyword">null</span>) { 
  <span class="hljs-keyword">return</span> {<span class="hljs-string">"ret"</span> : <span class="hljs-keyword">false</span>}; 
} 
<span class="hljs-comment">// 处理参数替换 </span>
<span class="hljs-keyword">if</span>(params != <span class="hljs-keyword">null</span>) { 
  params.forEach((k, v) =&gt; callApi = callApi.replaceAll(<span class="hljs-string">'{<span class="hljs-subst">$k</span>}'</span>, <span class="hljs-string">'<span class="hljs-subst">$v</span>'</span>)); 
} 
<span class="hljs-comment">// 调用服务端接口获取返回数据 </span>
<span class="hljs-keyword">try</span> { 
  Response response = <span class="hljs-keyword">await</span> Dio().<span class="hljs-keyword">get</span>( 
      callApi, 
      options: Options(responseType: ResponseType.json) 
  ); 
  <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; retInfo = 
    json.decode(response.toString()) <span class="hljs-keyword">as</span> <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt;; 
  <span class="hljs-keyword">return</span> retInfo; 
} <span class="hljs-keyword">catch</span> (e) { 
  <span class="hljs-keyword">return</span> {<span class="hljs-string">"ret"</span> : <span class="hljs-keyword">false</span>}; 
}
</code></pre>
<p data-nodeid="5064">第 2 行读取配置文件中的 api.json 数据（配置文件需要在 pubspec.yaml 中引入，具体查看源码中的第 55 和第 56 行），该 api.json 的部分数据如下：</p>
<pre class="lang-json" data-nodeid="5065"><code data-language="json">{ 
  <span class="hljs-attr">"recommendList"</span> : { 
    <span class="hljs-attr">"method"</span> : <span class="hljs-string">"get"</span>, 
    <span class="hljs-attr">"apiUrl"</span> : <span class="hljs-string">"https://www.fastmock.site/mock/978685eaf6950d1e2f0790f85cfdacaa/cgi-bin/recommend_list"</span>, 
    <span class="hljs-attr">"params"</span> : <span class="hljs-literal">null</span> 
  } 
}
</code></pre>
<p data-nodeid="5066">其中 JSON 部分就包括了协议名称，以及协议的请求方式和协议的 URL 以及具体的参数。<br>
在 get 方法中，获取到 api.json 数据后，再根据协议名称，获取到协议的 URL 。接下来经过一定的数据判断和参数处理，应用 dio 模块发起 get 网络请求。最后再使用 convert 库，将结构转化为 JSON 数据结构，并返回给到调用方。</p>
<h4 data-nodeid="5067">ApiContentIndex 实现</h4>
<p data-nodeid="5068">通用网络请求实现后，我们再看下具体的接口调用方的实现逻辑。接下来我们修改 ApiContentIndex 中的 getRecommendList 的代码，将原来的假数据转化为网络请求。因为是异步方法，因此还是需要使用 Future 和 async ，函数代码如下：</p>
<pre class="lang-dart" data-nodeid="5069"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/api_ret_info.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/struct/content_detail.dart'</span>; 
<span class="hljs-keyword">import</span> <span class="hljs-string">'package:two_you_friend/util/tools/call_server.dart'</span>; 
<span class="hljs-comment">/// <span class="markdown">获取内容详情接口 </span></span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ApiContentIndex</span> </span>{ 
  <span class="hljs-comment">/// <span class="markdown">拉取用户内容推荐帖子列表 </span></span>
  Future&lt;StructApiContentListRetInfo&gt; getRecommendList([lastId = <span class="hljs-keyword">null</span>]) <span class="hljs-keyword">async</span> { 

  } 
}
</code></pre>
<p data-nodeid="5070">代码第一部分还是引入相应的库，第二部分创建 ApiContentIndex 类，并创建 getRecommendList 函数，该函数异步返回 StructApiContentListRetInfo 数据结构，支持可选参数 lastId ，有 lastId 则拉取下一页，没有则拉取首页内容。接下来看下 getRecommendList 函数的具体逻辑，代码如下。</p>
<pre class="lang-dart" data-nodeid="5071"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">拉取用户内容推荐帖子列表 </span></span>
Future&lt;StructApiContentListRetInfo&gt; getRecommendList([lastId = <span class="hljs-keyword">null</span>]) <span class="hljs-keyword">async</span> { 
  <span class="hljs-keyword">if</span> (lastId != <span class="hljs-keyword">null</span>) { 
    <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; retJson = 
      <span class="hljs-keyword">await</span> CallServer.<span class="hljs-keyword">get</span>(<span class="hljs-string">'recommendListNext'</span>, {lastId: lastId}); 
    <span class="hljs-keyword">return</span> StructApiContentListRetInfo.fromJson(retJson); 
  } <span class="hljs-keyword">else</span> { 
    <span class="hljs-built_in">Map</span>&lt;<span class="hljs-built_in">String</span>, <span class="hljs-built_in">dynamic</span>&gt; retJson = 
    <span class="hljs-keyword">await</span> CallServer.<span class="hljs-keyword">get</span>(<span class="hljs-string">'recommendList'</span>); 
    <span class="hljs-keyword">return</span> StructApiContentListRetInfo.fromJson(retJson); 
  } 
}
</code></pre>
<p data-nodeid="5072">以上代码就比较简洁了，先根据 lastId 判断拉取首页还是拉取下一页，如果拉取首页，则调用 recommendList 协议，如果拉取下一页，则调用 recommendListNext 协议。使用 CallServer.get 方法与服务端交互，得到返回数据结构后，调用 StructApiContentListRetInfo.fromJson 转化为 StructApiContentListRetInfo 数据结构，这样就实现了具体的 API 协议，最后我们再来看下在页面中调用 api 的使用方法。</p>
<h4 data-nodeid="5073">HomePageIndex</h4>
<p data-nodeid="5074">因为 ApiContentIndex 协议是在 HomePageIndex 这个类中调用，我们就来看下这块的处理逻辑，相同部分我们就不过多介绍。</p>
<pre class="lang-dart" data-nodeid="5075"><code data-language="dart"><span class="hljs-comment">/// <span class="markdown">处理首次拉取和刷新数据获取动作 </span></span>
<span class="hljs-keyword">void</span> setFirstPage() { 
  ApiContentIndex().getRecommendList().then((retInfo){ 
    <span class="hljs-keyword">if</span> (retInfo.ret != <span class="hljs-number">0</span>) { 
      <span class="hljs-comment">// 判断返回是否正确 </span>
      error = <span class="hljs-keyword">true</span>; 
      <span class="hljs-keyword">return</span>; 
    } 
    setState(() { 
      error = <span class="hljs-keyword">false</span>; 
      contentList = retInfo.data; 
      hasMore = retInfo.hasMore; 
      isLoading = <span class="hljs-keyword">false</span>; 
      lastId = retInfo.lastId; 
    }); 
  }); 
}
</code></pre>
<p data-nodeid="5076">在 setFirstPage 中调用类 ApiContentIndex 中的异步方法 getRecommendList ，在 getRecommendList 回调中成功获取数据后使用 setState 更新页面状态。由于网络请求有时间延迟，因此在页面刚加载时，需要使用 loading 组件，需要更改原来的 build 方法，修改部分如下：</p>
<pre class="lang-dart" data-nodeid="5077"><code data-language="dart"><span class="hljs-keyword">if</span> (error) { 
  <span class="hljs-keyword">return</span> CommonError(action: <span class="hljs-keyword">this</span>.setFirstPage); 
} 
<span class="hljs-keyword">if</span>(contentList == <span class="hljs-keyword">null</span>){ 
  <span class="hljs-keyword">return</span> Loading(); 
}
</code></pre>
<p data-nodeid="5078">主要是第 4 行，增加了对数据的判断，如果为空则显示 loading 组件内容，具体效果如下图 3 所示。</p>
<p data-nodeid="5079"><img src="https://s0.lgstatic.com/i/image/M00/3A/45/CgqCHl8hOHuANKsHACSrYreWeyI011.gif" alt="20200717_233752.gif" data-nodeid="5234"></p>
<div data-nodeid="5080"><p style="text-align:center">图 3 网络请求 loading 效果</p></div>
<p data-nodeid="5081">以上就完成了 ApiContentIndex 部分的 getRecommendList 逻辑，其他代码逻辑基本相似，具体大家可以参考 github 上的源码。</p>
<h3 data-nodeid="5082">总结</h3>
<p data-nodeid="5083">本课时介绍了 APP 常用的三种网络传输协议序列化方式，其次介绍了 Flutter 与服务端的网络通信方法，并且通过传输协议与服务端进行交互获取数据。学完本课时后要着重掌握 JSON 和 Protocol Buffer 的使用方法，其次掌握网络请求库 CallServer 的实现原理。</p>
<p data-nodeid="5084">下一课时我们将整理我们在 Two You APP 研发过程中所涉及的布局逻辑，介绍在 Flutter 中常见的一些布局原理和思想，并用此理论来完善我们 APP 内的“客人态页面” 的功能。</p>
<p data-nodeid="5085" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="5241">点击此链接查看本课时源码</a></p>

---

### 精选评论

##### *斌：
> StructApiContentListRetInfo.fromJson(Map">, dynamic json)    : ret = json['ret'] as int,    message = json['message'] as String,      hasMore = json['hasMore'] as bool,      lastId = json['lastId'] as String,    data = getContentDetailList(json['data'] as List);老师这个fromJson 后面跟：不理解啥意思

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个目的是将 JSON 数据结构转化为 Struct 结构，fromJson 的作用就是这个目的，在 Dart 中是一个强数据类型的语言，不建议直接用动态数据类型 JSON，而是将 JSON 转化为 Struct 结构，这两个之间的桥梁就是 fromJson。

##### *艺：
> 第一个lastId应该是作为key的应该要加隐号吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以不用加的呢，在 dart 语法中 map 结构的 key 可以不加引号，加上也没什么问题。

##### *蛋：
> 老师，报异常：[VERBOSE-2:ui_dart_state.cc(166)] Unhandled Exception: Unable to load asset: assets/json/api.json

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你检查下，有没有按照我说的步骤，特别是需要在pubspec.yaml中增加配置文件引入部分。其次你看下这个路径下是否存在api.json文件。

