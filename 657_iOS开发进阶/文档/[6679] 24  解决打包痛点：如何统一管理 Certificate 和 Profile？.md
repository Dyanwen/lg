<p data-nodeid="1279" class="">一个优秀的 iOS 开发者所需要做的工作不仅仅是编写代码那么简单，还要管理证书（Certificates）和 Provisioning Profile、打包和签名 App、上架与分发等。你如果做过这些操作的话应该知道，单纯通过手工的方式来完成，每个步骤都需要花费大量的时间，而且十分容易出错。</p>
<p data-nodeid="1280">那有没有什么办法能帮我们节省这些手工操作的时间呢？答案当然是肯定的。我们可以<strong data-nodeid="1402">通过 fastlane 来自动化这些操作</strong>，所以，在这一讲中我们就来介绍下如何通过 fastlane 来统一管理证书和 Provisioning Profiles。</p>
<h3 data-nodeid="1281">什么是证书和 Provisioning Profile</h3>
<p data-nodeid="1282">刚接触 iOS 的开发者可能都会有一个疑惑：为什么在 iOS 开发过程中需要管理私钥、证书、Provisioning Profile 以及设备列表等信息呢？</p>
<p data-nodeid="1283">这是因为<strong data-nodeid="1410">苹果要给 App 的终端用户提供安全和稳定的体验</strong>。而要达到这一效果，苹果就得要求所有开发者在用户安装之前必须为 App 进行打包和签名。有了这些签名，苹果就知道这些 App 到底是谁开发的，签名以后 App 是否被修改过。</p>
<p data-nodeid="1284">这里的打包和签名操作就涉及私钥、证书和 Provisioning Profile 等组件，我们可以结合下面这张图看看这些组件之间的关系：</p>
<p data-nodeid="1285"><img src="https://s0.lgstatic.com/i/image6/M00/3E/F6/Cgp9HWCbpROAFzsfAAbfDL-IX24841.png" alt="图片4.png" data-nodeid="1414"></p>
<p data-nodeid="1286">那这些组件到底都有什么作用呢？下面我们来分别说明下。</p>
<ul data-nodeid="1287">
<li data-nodeid="1288">
<p data-nodeid="1289"><strong data-nodeid="1420">苹果证书机构</strong>。世界上有好多证书机构（CA），但当我们通过 App Store Connect 发布 App 的时候，苹果公司只认它自己的证书机构。因为苹果证书机构归苹果公司所有，所以苹果公司对安装到设备上的所有 iOS App 都有最终的控制权。</p>
</li>
<li data-nodeid="1290">
<p data-nodeid="1291"><strong data-nodeid="1425">私钥</strong>。这是生成签名证书所需的私钥文件，通常是一个后缀名为 .p12 的文件。私钥是证明我们身份的唯一信息源，假如丢失了这个私钥，那其他人就能伪装成我们了，非常不安全。当我们手工生成证书时，会通过 Keychain Access 程序生成一个后缀为 .certSigningRequest 的 Certificate Signing Request 文件和私钥文件，然后把 .certSigningRequest 文件上传到苹果开发者网站，苹果公司就可以通过这个请求，并使用苹果证书机构来为我们发行一个证书。</p>
</li>
<li data-nodeid="1292">
<p data-nodeid="1293"><strong data-nodeid="1430">签名证书</strong>。签名证书是由苹果证书机构通过提供的 .certSigningRequest 文件所签发的，因此苹果公司知道这个证书的所有人。苹果公司会把这个证书作为签名主体。签名证书通常是一个后缀名为 .cer 的文件，该 .cer 文件包含了开发者 ID、团队 ID 和公钥信息。</p>
</li>
<li data-nodeid="1294">
<p data-nodeid="1295"><strong data-nodeid="1435">发布渠道</strong>。我们可以把 App 通过不同渠道发布出去，目前支持的渠道有 Development、Ad Hoc、Enterprise 和 App Store。当我们在 Xcode 上把 App 部署到设备进行 Debug 时，一般会使用 Development 渠道。当我们把 App 分发给内部测试用户时，可以使用 Ad Hoc 渠道。如果开发企业内的 App，可以使用 Enterprise 渠道来发布。而对于要上传到 App Store 的 App，就必须使用 App Store 渠道了。</p>
</li>
<li data-nodeid="1296">
<p data-nodeid="1297"><strong data-nodeid="1440">App ID</strong>。每个 App 都有唯一的 ID。根据不同的用途，我们为 Moments App 建立了三个 App ID，分别用于开发与调试、内部测试和上架 App Store。</p>
</li>
<li data-nodeid="1298">
<p data-nodeid="1299"><strong data-nodeid="1445">设备列表</strong>。当我们通过 Ad Hoc 渠道来发布 App 时，要把需要安装 App 的设备添加到设备列表中，只有在设备列表中的设备才能安装 Ad Hoc 渠道的 App。</p>
</li>
<li data-nodeid="1300">
<p data-nodeid="1301"><strong data-nodeid="1450">Provisioning Profile</strong>。有了证书以后，我们可以为不同的 App ID 以及不同的发布渠道来生成不同的 Provisioning profile，通常是一个后缀名为 .mobileprovision 或 .provisionprofile 的文件。该文件包含 App ID 所指向的 Entitlements 信息，以及发布渠道、团队 ID 和设备列表信息。我们通常为不同用途的 App 生成不同的 Provisioning Profile，例如我们为 Moments App 的 App Store 版本生成一个 Provisioning Profile，然后再为 Moments App 的 Internal 版本生成另外一个 Provisioning Profile。</p>
</li>
</ul>
<h3 data-nodeid="1302">搭建管理证书和 Provisioning Profile 的环境</h3>
<p data-nodeid="1303">假如你手工生成过私钥、证书和 Provisioning Profile 文件，并在苹果开发者网站上进行过上传、下载和安装的话，就知道这些操作过程有多麻烦。</p>
<ul data-nodeid="1304">
<li data-nodeid="1305">
<p data-nodeid="1306">有些团队会为每个成员生成多个不同的证书来进行签名，这将导致大量证书和 Provisioning Profile 文件的出现，十分难管理。</p>
</li>
<li data-nodeid="1307">
<p data-nodeid="1308">证书都是有期限的，每次延展期限都需要手工更新所有的 Provisioning Profiles。</p>
</li>
<li data-nodeid="1309">
<p data-nodeid="1310">当添加新增设备时，也需要更新 Ad Hoc 的 Provisioning Profiles。</p>
</li>
<li data-nodeid="1311">
<p data-nodeid="1312">当搭建 CI 的时候，又需要花大量时间来下载、安装私钥、证书和 Provisioning Profiles 文件。</p>
</li>
</ul>
<p data-nodeid="1313">那有没有什么办法来简化证书和 Provisioning Profiles 的管理工作呢？幸运的是<strong data-nodeid="1462">fastlane 为我们提供了一个名叫 match 的 Action 来为整个团队统一管理并共享所有证书和 Provisioning Profile</strong>。</p>
<p data-nodeid="1314">下面我们就来看一下如何使用 fastlane 的 match Action 搭建所需的环境吧。</p>
<h4 data-nodeid="1315">建 GitHub 私有 Repo</h4>
<p data-nodeid="1316">为了把证书共享给整个团队使用，fastlane match 需要把私钥和证书保存在云端的存储服务上。目前支持的云存储服务有亚马逊的 S3、谷歌云和微软的 Azure 等。但<strong data-nodeid="1470">我推荐使用 GitHub 私有 Repo 来存储私钥和证书，因为 GitHub 私有 Repo 是免费的，而且有详细的修改历史</strong>。Moments App 的证书就保存在 GitHub 的私有 Repo 里面，下面我们讲一下如何搭建 GitHub 私有 Repo。</p>
<p data-nodeid="1317">我们可以点击 GitHub 网站右上角的加号（+）按钮，然后选择 New repository 菜单来新建私有 Repo。因为该 Repo 用于签名，所以我会以“&lt;项目名称&gt;-codesign”的方式来命名，例如叫 moments-codesign。具体页面情况如下图所示：</p>
<p data-nodeid="1318"><img src="https://s0.lgstatic.com/i/image6/M00/3D/C3/CioPOWCWMc2ASbGgAAEeGAfOCoA064.png" alt="Drawing 1.png" data-nodeid="1476"></p>
<p data-nodeid="1319">这里需要注意：<strong data-nodeid="1482">我们必须把 Repo 设置为 Private，因为该 Repo 保存了私钥等关键信息</strong>，一旦设置为 Public 的话，所有人都可以访问它了。</p>
<h4 data-nodeid="1320">生成 GitHub Access Token</h4>
<p data-nodeid="1321"><strong data-nodeid="1487">那怎样才能让整个团队都能访问这个私有 Repo 呢？答案是使用 GitHub Access Token。</strong></p>
<p data-nodeid="1322">我推荐的做法是为每一个 App 新建一个 GitHub 账户，例如新建一个叫作 momentsci 的账户，然后把该账户添加到私有 Repo 的贡献者列表里面。如下图所示：</p>
<p data-nodeid="1323"><img src="https://s0.lgstatic.com/i/image6/M00/3D/C3/CioPOWCWMdSAYvK7AADj0dRNEHo360.png" alt="Drawing 2.png" data-nodeid="1491"></p>
<p data-nodeid="1324">这样子，momentsci 用户就能访问和更新该私有 Repo 了。</p>
<p data-nodeid="1325">下一步是为 momentsci 用户生成 GitHub Access Token。当我们通过 momentsci 登录到 GitHub 以后，点击 Settings -&gt; Developer settings -&gt; Personal access tokens 来打开来配置页面，接着再点击 Generate new token 按钮，在 Note 输入框填写 Token 的用途，比如写上“用于 Moments App 的 CI”，然后在 Select scopes 选上 repo，如下图所示：</p>
<p data-nodeid="1326"><img src="https://s0.lgstatic.com/i/image6/M00/3D/C3/CioPOWCWMdyAGRYeAAGmH0NcYDk620.png" alt="Drawing 3.png" data-nodeid="1496"></p>
<p data-nodeid="1327">因为我们选择了 Full controll of private repositories（能完全控制所有私有 Repo），所以使用该 Token 的应用程序（例如 fastlane）就有权限访问 momentsci 用户所能访问的所有 Repo，并且能 push commit 到这些 Repo 去。当我们点击 Generate token 按钮以后就生成一个如下图所示的 Token：</p>
<p data-nodeid="1328"><img src="https://s0.lgstatic.com/i/image6/M01/3E/F6/Cgp9HWCbpVKAIJRSAAQzSVGBgVk131.png" alt="图片5.png" data-nodeid="1500"></p>
<p data-nodeid="1329">这里需要注意，我们<strong data-nodeid="1506">一定要好好保存这个 Token</strong>，因为一旦关闭该页面以后就无法再从 GitHub 上找到该 Token 了。为了使得团队所有人都可以使用到这个 Token，我推荐把它存放在团队密码共享服务里面，目前比较流行的密码共享服务有 LastPass、OnePassword 等。</p>
<p data-nodeid="1330">有了这个 Token 以后，我们还需要生成一个 BASE64 字符串提供给 fastlane 使用，命令如下：</p>
<pre class="lang-java" data-nodeid="1331"><code data-language="java">$&gt; echo -n your_github_username:your_personal_access_token | base64
</code></pre>
<p data-nodeid="1332">不过需要把 your_github_username 替换为 GitHub 用户名，例如 momentsci 用户，然后把 your_personal_access_token 替换成刚才所生成的 Token。</p>
<p data-nodeid="1333">接着就可以在 Shell 里把 BASE64 赋值给环境变量<code data-backticks="1" data-nodeid="1520">MATCH_GIT_BASIC_AUTHORIZATION</code>，如下所示：</p>
<pre class="lang-java" data-nodeid="1334"><code data-language="java">$&gt; export MATCH_GIT_BASIC_AUTHORIZATION=&lt;YOUR BASE64 KEY&gt;
</code></pre>
<p data-nodeid="1335">为了提高安全性，我们还可以配置环境变量<code data-backticks="1" data-nodeid="1523">MATCH_PASSWORD</code>来加密私钥、证书和 Provisioning Profile 文件。但是需要注意：<strong data-nodeid="1529">一定要记住这个密码，因为使用这些文件的机器都需要使用到该密码</strong>。</p>
<pre class="lang-java" data-nodeid="1336"><code data-language="java">$&gt; export MATCH_PASSWORD=&lt;YOUR MATCH PASSWORD&gt;
</code></pre>
<h4 data-nodeid="1337">生成 App Store Connect API Key</h4>
<p data-nodeid="1338">因为生成证书和 Provisioning Profile 的过程需要与苹果开发者网站进行交互，所以 fastlane 也需要具备访问苹果开发者网站的权限。</p>
<p data-nodeid="1339">目前 fastlane 提供了几种办法来访问苹果开发者网站，例如，输入登录所需的用户名和密码等。但<strong data-nodeid="1537">我推荐使用 App Store Connect API Key 的方式</strong>，因为 API Key 既能有效控制访问权限，也可以随时让该 Key 失效。</p>
<p data-nodeid="1340">我们可以在 App Store Connect 网站上生成该 Key，在网站上选择 Users and Access -&gt; Keys -&gt; App Store Connect API，然后点击加号（+）来生成 Key，会弹出下面的输入框：</p>
<p data-nodeid="1341"><img src="https://s0.lgstatic.com/i/image6/M01/3E/F7/Cgp9HWCbpZiAbIChAAC_Yq3o-fE466.png" alt="图片7.png" data-nodeid="1541"></p>
<p data-nodeid="1342">Key 的名称可以填写其用途，例如使用到 CI 上我们就填 “Moments CI”，然后在 Access 里选择 App Manager。需要注意：<strong data-nodeid="1547">必须选择 App Manager 以上的权限</strong>，因为使用 App Manager 以下的权限，fastlane 在执行过程中会出错。这是 fastlane 的一个已知的问题，假如以后解决了该问题，我们就可以选择 Developer 权限，原则上是该 Key 的 Access 权限越低就越安全。</p>
<p data-nodeid="1343">当 Key 生成完毕后，我们需要把它保存起来，并在 Shell 里把该 Key 赋值给环境变量<code data-backticks="1" data-nodeid="1549">APP_STORE_CONNECT_API_CONTENT</code>，如下所示：</p>
<pre class="lang-java" data-nodeid="1344"><code data-language="java">$&gt; export APP_STORE_CONNECT_API_CONTENT=&lt;App Store Connect API&gt;
</code></pre>
<p data-nodeid="1345">到这里，管理证书和 Provisioning Profile 的环境就配置完了。配置的步骤虽然有点多，但是<strong data-nodeid="1556">每个项目只需配置一次就好了，其他项目成员无须重复配置</strong>。为了进一步简化环境变量的赋值操作，我推荐在项目根目录下建立一个名叫 local.keys 的文件，然后把所有环境变量都放在该文件里面，如下所示：</p>
<pre class="lang-java" data-nodeid="1346"><code data-language="java">APP_STORE_CONNECT_API_CONTENT=&lt;App Store Connect API <span class="hljs-keyword">for</span> an App Manager&gt;
GITHUB_API_TOKEN=&lt;GitHub API token <span class="hljs-keyword">for</span> accessing the <span class="hljs-keyword">private</span> repo <span class="hljs-keyword">for</span> certificates and provisioning profiles&gt;
MATCH_PASSWORD=&lt;Password <span class="hljs-keyword">for</span> certificates <span class="hljs-keyword">for</span> App signing on GitHub <span class="hljs-keyword">private</span> repo&gt;
</code></pre>
<p data-nodeid="1347">接着在根目录执行以下的命令：</p>
<pre class="lang-java" data-nodeid="1348"><code data-language="java">$&gt; source ./scripts/export_env.sh
</code></pre>
<p data-nodeid="1349">这样就能把所有环境变量一次性导入当前的 Shell 里面，不过注意，这里需要使用<code data-backticks="1" data-nodeid="1559">source</code>命令，否则环境变量只会导出到子 Shell 里面。</p>
<p data-nodeid="1350">这里还需要提醒一下，因为我们不应该把机密信息上传到 Git 服务器上，所以该 local.keys 文件需要配置到 .gitignore 文件里面。</p>
<h3 data-nodeid="1351">使用 fastlane 管理证书和 Provisioning Profile</h3>
<p data-nodeid="1352">有了上述的环境搭建与配置，我们就可以使用 fastlane 来统一管理证书和 Provisioning Profile 了。</p>
<h4 data-nodeid="1353">生成证书和 Provisioning Profile</h4>
<p data-nodeid="1354">第一步是生成证书和 Provisioning Profile，每个项目也只需执行一次这样的操作。</p>
<p data-nodeid="1355">为了简化，我把生成证书和 Profile 的操作都封装在<code data-backticks="1" data-nodeid="1567">create_new_profiles</code>Lane 里面，只需要执行<code data-backticks="1" data-nodeid="1569">bundle exec fastlane create_new_profiles</code>命令即可，该 Lane 的具体代码如下：</p>
<pre class="lang-ruby" data-nodeid="1356"><code data-language="ruby">desc <span class="hljs-string">"Create all new provisioning profiles managed by fastlane match"</span>
lane <span class="hljs-symbol">:create_new_profiles</span> <span class="hljs-keyword">do</span>
  api_key = get_app_store_connect_api_key
  keychain_name = <span class="hljs-string">"TemporaryKeychain"</span>
  keychain_password = <span class="hljs-string">"TemporaryKeychainPassword"</span>
  create_keychain(
    <span class="hljs-symbol">name:</span> keychain_name,
    <span class="hljs-symbol">password:</span> keychain_password,
    <span class="hljs-symbol">default_keychain:</span> <span class="hljs-literal">false</span>,
    <span class="hljs-symbol">timeout:</span> <span class="hljs-number">3600</span>,
    <span class="hljs-symbol">unlock:</span> <span class="hljs-literal">true</span>,
  )
  match(
    <span class="hljs-symbol">type:</span> <span class="hljs-string">"adhoc"</span>,
    <span class="hljs-symbol">keychain_name:</span> keychain_name,
    <span class="hljs-symbol">keychain_password:</span> keychain_password,
    <span class="hljs-symbol">storage_mode:</span> <span class="hljs-string">"git"</span>,
    <span class="hljs-symbol">git_url:</span> <span class="hljs-string">"https://github.com/JakeLin/moments-codesign"</span>,
    <span class="hljs-symbol">app_identifier:</span> <span class="hljs-string">"com.ibanimatable.moments.internal"</span>,
    <span class="hljs-symbol">team_id:</span> <span class="hljs-string">"6HLFCRTYQU"</span>,
    <span class="hljs-symbol">api_key:</span> api_key
  )
  match(
    <span class="hljs-symbol">type:</span> <span class="hljs-string">"appstore"</span>,
    <span class="hljs-symbol">keychain_name:</span> keychain_name,
    <span class="hljs-symbol">keychain_password:</span> keychain_password,
    <span class="hljs-symbol">storage_mode:</span> <span class="hljs-string">"git"</span>,
    <span class="hljs-symbol">git_url:</span> <span class="hljs-string">"https://github.com/JakeLin/moments-codesign"</span>,
    <span class="hljs-symbol">app_identifier:</span> <span class="hljs-string">"com.ibanimatable.moments"</span>,
    <span class="hljs-symbol">team_id:</span> <span class="hljs-string">"6HLFCRTYQU"</span>,
    <span class="hljs-symbol">api_key:</span> api_key
  )
<span class="hljs-keyword">end</span>
</code></pre>
<p data-nodeid="1357">该 Lane 主要由三部分组成。</p>
<p data-nodeid="1358">第一部分是调用<code data-backticks="1" data-nodeid="1573">create_keychain</code>Action 来生成 Keychain。因为 fastlane 所生成的私钥和证书都需要保存在 Keychain 里，所以我们要生成一个 Keychain 来保存它们。为了不影响默认的 Keychain，我们把<code data-backticks="1" data-nodeid="1575">false</code>传递给<code data-backticks="1" data-nodeid="1577">default_keychain</code>参数，表示生成的 Keychain 不是默认的 Keychain。</p>
<p data-nodeid="1359">第二部分是通过指定 Ad Hoc 作为发布渠道来为 Internal App 生成证书和 Provisioning Profile，并把它们上传到 GitHub 私有 Repo，这样我们就能使用这个 Provisioning Profile 为 Internal App 进行签名和打包。</p>
<p data-nodeid="1360">第三部分与第二部分非常类似，也是用于生成证书和 Provisioning Profile。不同的是它生成了发布渠道为 Appstore 类型的 Provisioning Profile，有了该 Provisioning Profile，我们就能为 AppStore 版本的 App 进行签名和打包。</p>
<p data-nodeid="1361">你可能发现，我们调用了私有 Lane<code data-backticks="1" data-nodeid="1582">get_app_store_connect_api_key</code>来获取<code data-backticks="1" data-nodeid="1584">api_key</code>变量的值。该私有 Lane 的定义如下：</p>
<pre class="lang-ruby" data-nodeid="1362"><code data-language="ruby">desc <span class="hljs-string">'Get App Store Connect API key'</span>
  private_lane <span class="hljs-symbol">:get_app_store_connect_api_key</span> <span class="hljs-keyword">do</span>
    key_content = ENV[<span class="hljs-string">"APP_STORE_CONNECT_API_CONTENT"</span>]
    api_key = app_store_connect_api_key(
      <span class="hljs-symbol">key_id:</span> <span class="hljs-string">"D9B979RR69"</span>,
      <span class="hljs-symbol">issuer_id:</span> <span class="hljs-string">"69a6de7b-13fb-47e3-e053-5b8c7c11a4d1"</span>,
      <span class="hljs-symbol">key_content:</span> <span class="hljs-string">"-----BEGIN EC PRIVATE KEY-----\n"</span> + key_content + <span class="hljs-string">"\n-----END EC PRIVATE KEY-----"</span>,
      <span class="hljs-symbol">duration:</span> <span class="hljs-number">1200</span>,
      <span class="hljs-symbol">in_house:</span> <span class="hljs-literal">false</span>
    )
    api_key 
  <span class="hljs-keyword">end</span>
</code></pre>
<p data-nodeid="1363">该私有 Lane 从环境变量中读取了<code data-backticks="1" data-nodeid="1587">APP_STORE_CONNECT_API_CONTENT</code>的值，然后通过调用<code data-backticks="1" data-nodeid="1589">app_store_connect_api_key</code>Action 来获取临时的 App Store Connect API Key。其中，<code data-backticks="1" data-nodeid="1591">key_id</code>和<code data-backticks="1" data-nodeid="1593">issuer_id</code>的值都可以在 App Store Connect 的 Keys 配置页面上找到。</p>
<p data-nodeid="1364">如果你没有为 GitHub 配置全局的用户名和邮箱，那么在执行<code data-backticks="1" data-nodeid="1596">bundle exec fastlane create_new_profiles</code>命令时可能会出错。你可以通过下面的命令来解决这个问题，在命令执行完之后还可通过<code data-backticks="1" data-nodeid="1598">git config --global --edit</code>命令把这些配置删掉。</p>
<pre class="lang-java" data-nodeid="1365"><code data-language="java">$&gt; git config --global user.email <span class="hljs-string">"MomentsCI@lagou.com"</span>
$&gt; git config --global user.name <span class="hljs-string">"Moments CI"</span>
</code></pre>
<p data-nodeid="1366">当<code data-backticks="1" data-nodeid="1601">create_new_profiles</code>命令成功执行以后，你可以在私有 Repo 上看到两个新的文件夹，如下图所示：</p>
<p data-nodeid="1367"><img src="https://s0.lgstatic.com/i/image6/M01/3E/F7/Cgp9HWCbpdWASXy9AAJKo3Mqtpg236.png" alt="图片8.png" data-nodeid="1605"></p>
<p data-nodeid="1368">其中，<strong data-nodeid="1611">certs 文件夹用于保存私钥（.p12）和证书（.cer）文件，而 profiles 文件夹则用来保存 adhoc 和 appstore 两个 Provisioning Profile 文件</strong>。</p>
<p data-nodeid="1369">你也可以在苹果开发者网站查看新的证书文件：</p>
<p data-nodeid="1370"><img src="https://s0.lgstatic.com/i/image6/M01/3E/F7/Cgp9HWCbpfqAKc17AAFyb88k84o360.png" alt="图片9.png" data-nodeid="1615"></p>
<p data-nodeid="1371">同时还可以看到 Provisioning Profile 文件：</p>
<p data-nodeid="1372"><img src="https://s0.lgstatic.com/i/image6/M00/3E/FF/CioPOWCbpheAOxNSAAFxbPkMv1o580.png" alt="图片10.png" data-nodeid="1619"></p>
<p data-nodeid="1373">除此之外，你还可以在 Keychain App 里面找到新增的私钥和证书，如下图所示：</p>
<p data-nodeid="1374"><img src="https://s0.lgstatic.com/i/image6/M00/3D/C3/CioPOWCWMyeAQN7wAAOKWo3am2o738.png" alt="Drawing 9.png" data-nodeid="1623"></p>
<h4 data-nodeid="1375">下载证书和 Provisioning Profile</h4>
<p data-nodeid="1376">一个项目只需要执行一次生成证书和 Provisioning Profile 的操作，其他团队成员可通过<code data-backticks="1" data-nodeid="1626">bundle exec fastlane download_profiles</code>命令来下载证书和 Provisioning Profile。该 Lane 的代码如下：</p>
<pre class="lang-ruby" data-nodeid="1377"><code data-language="ruby">desc <span class="hljs-string">"Download certificates and profiles"</span>
lane <span class="hljs-symbol">:download_profiles</span> <span class="hljs-keyword">do</span>
  keychain_name = <span class="hljs-string">"TemporaryKeychain"</span>
  keychain_password = <span class="hljs-string">"TemporaryKeychainPassword"</span>
  create_keychain(
    <span class="hljs-symbol">name:</span> keychain_name,
    <span class="hljs-symbol">password:</span> keychain_password,
    <span class="hljs-symbol">default_keychain:</span> <span class="hljs-literal">false</span>,
    <span class="hljs-symbol">timeout:</span> <span class="hljs-number">3600</span>,
    <span class="hljs-symbol">unlock:</span> <span class="hljs-literal">true</span>,
  )
  match(
    <span class="hljs-symbol">type:</span> <span class="hljs-string">"adhoc"</span>,
    <span class="hljs-symbol">readonly:</span> <span class="hljs-literal">true</span>,
    <span class="hljs-symbol">keychain_name:</span> keychain_name,
    <span class="hljs-symbol">keychain_password:</span> keychain_password,
    <span class="hljs-symbol">storage_mode:</span> <span class="hljs-string">"git"</span>,
    <span class="hljs-symbol">git_url:</span> <span class="hljs-string">"https://github.com/JakeLin/moments-codesign"</span>,
    <span class="hljs-symbol">app_identifier:</span> <span class="hljs-string">"com.ibanimatable.moments.internal"</span>,
    <span class="hljs-symbol">team_id:</span> <span class="hljs-string">"6HLFCRTYQU"</span>
  )
  match(
    <span class="hljs-symbol">type:</span> <span class="hljs-string">"appstore"</span>,
    <span class="hljs-symbol">readonly:</span> <span class="hljs-literal">true</span>,
    <span class="hljs-symbol">keychain_name:</span> keychain_name,
    <span class="hljs-symbol">keychain_password:</span> keychain_password,
    <span class="hljs-symbol">storage_mode:</span> <span class="hljs-string">"git"</span>,
    <span class="hljs-symbol">git_url:</span> <span class="hljs-string">"https://github.com/JakeLin/moments-codesign"</span>,
    <span class="hljs-symbol">app_identifier:</span> <span class="hljs-string">"com.ibanimatable.moments"</span>,
    <span class="hljs-symbol">team_id:</span> <span class="hljs-string">"6HLFCRTYQU"</span>
  )
<span class="hljs-keyword">end</span>
</code></pre>
<p data-nodeid="1378">你会发现<code data-backticks="1" data-nodeid="1629">download_profiles</code>和<code data-backticks="1" data-nodeid="1631">create_new_profiles</code>两个 Lane 的实现非常类似，都是由三部分组成，包括生成 Keychain、下载 Internal App 的证书和 Provisioning Profile 以及 AppStore 版本 App 的证书和 Provisioning Profile。不同的地方是<code data-backticks="1" data-nodeid="1633">download_profiles</code>Lane 不需要更新 App Store Connect，所以无须使用 App Store Connect 的 API Key；并且<code data-backticks="1" data-nodeid="1635">download_profiles</code>Lane 也不需要更新私有 Repo 的内容，所以在调用<code data-backticks="1" data-nodeid="1637">match</code>Action 时，我们会把<code data-backticks="1" data-nodeid="1639">true</code>传递给<code data-backticks="1" data-nodeid="1641">readonly</code>参数。</p>
<h4 data-nodeid="1379">新增设备</h4>
<p data-nodeid="1380">当我们通过 Ad Hoc 的方式来分发 App 时，必须把需要安装 App 的设备 ID 都添加到设备列表里面，你可以在苹果开发者网站的“Certificates, Identifiers &amp; Profiles”的 Devices 下查看所有设备信息。如下图所示：</p>
<p data-nodeid="1381"><img src="https://s0.lgstatic.com/i/image6/M00/3E/FF/CioPOWCbpj2AYt-xAAFbLA-2M_0002.png" alt="图片11.png" data-nodeid="1649"></p>
<p data-nodeid="1382">但是手工更新设备列表的操作比较麻烦，而且更新完以后还需要再更新 Provisioning Profile。幸运的是 fastlane 能帮我们自动化这些操作，我们把这些操作都封装在<code data-backticks="1" data-nodeid="1651">add_device</code>Lane 里面，具体代码如下：</p>
<pre class="lang-ruby" data-nodeid="1383"><code data-language="ruby">desc <span class="hljs-string">"Add a new device to provisioning profile"</span>
lane <span class="hljs-symbol">:add_device</span> <span class="hljs-keyword">do</span> <span class="hljs-params">|options|</span>
  name = options[<span class="hljs-symbol">:name</span>]
  udid = options[<span class="hljs-symbol">:udid</span>]
  <span class="hljs-comment"># Add to App Store Connect</span>
  api_key = get_app_store_connect_api_key
  register_device(
    <span class="hljs-symbol">name:</span> name,
    <span class="hljs-symbol">udid:</span> udid,
    <span class="hljs-symbol">team_id:</span> <span class="hljs-string">"6HLFCRTYQU"</span>,
    <span class="hljs-symbol">api_key:</span> api_key
  )
  <span class="hljs-comment"># Update the profiles to Git private repo</span>
  match(
    <span class="hljs-symbol">type:</span> <span class="hljs-string">"adhoc"</span>,
    <span class="hljs-symbol">force:</span> <span class="hljs-literal">true</span>,
    <span class="hljs-symbol">storage_mode:</span> <span class="hljs-string">"git"</span>,
    <span class="hljs-symbol">git_url:</span> <span class="hljs-string">"https://github.com/JakeLin/moments-codesign"</span>,
    <span class="hljs-symbol">app_identifier:</span> <span class="hljs-string">"com.ibanimatable.moments.internal"</span>,
    <span class="hljs-symbol">team_id:</span> <span class="hljs-string">"6HLFCRTYQU"</span>,
    <span class="hljs-symbol">api_key:</span> api_key
  )
<span class="hljs-keyword">end</span>
</code></pre>
<p data-nodeid="1384">首先调用<code data-backticks="1" data-nodeid="1654">register_device</code>Action 把设备更新到苹果开发者网站上的设备列表里面，然后把<code data-backticks="1" data-nodeid="1656">true</code>传递给<code data-backticks="1" data-nodeid="1658">force</code>参数来调用<code data-backticks="1" data-nodeid="1660">match</code>Action，这个操作能强制更新 Ad Hoc 的 Provisioning Profile 并上传到私有 Repo 里。这样当其他机器在调用<code data-backticks="1" data-nodeid="1662">download_profiles</code>命令的时候，就能获取最新的 Provisioning Profile 了。</p>
<h3 data-nodeid="1385">总结</h3>
<p data-nodeid="1386">在这一讲中，我们讲述了如何使用 fastlane 的 match Action 来帮我们统一管理签名和打包所需的私钥、证书和 Provisioning Profile 文件。</p>
<p data-nodeid="1387">在实际项目中，我们只需要<strong data-nodeid="1677">一次性完成搭建的任务</strong>，例如生成私钥 Repo、导出 Github Access Token 和 App Store Connect API Key，以及调用<code data-backticks="1" data-nodeid="1671">create_new_profiles</code>来生成所需的证书和 Provisioning Profile。其他团队成员和 CI 服务器就可以通过调用<code data-backticks="1" data-nodeid="1673">download_profiles</code>来下载证书。当需要为 Ad Hoc 发布渠道添加新设备时，只需要执行<code data-backticks="1" data-nodeid="1675">add_device</code>即可。</p>
<p data-nodeid="14575" class="te-preview-highlight">有了<code data-backticks="1" data-nodeid="14577">download_profiles</code>和<code data-backticks="1" data-nodeid="14579">add_device</code>等命令，团队里任何人都可以轻松地下载打包和签名所需的私钥、证书和 Provisioning Profile 文件，无须手工使用 Keychain Access 程序来管理私钥，无须登录到苹果开发者网站下载和安装证书，无须到苹果开发者网站上手工添加设备，无须重新生成 Provisioning Profile 等。这样能减少大量无聊而且容易出错的手工操作工作，让我们把有效的时间都花在功能开发与迭代上。</p>
















<p data-nodeid="1389"><strong data-nodeid="1694">思考题</strong></p>
<blockquote data-nodeid="1390">
<p data-nodeid="1391">在 Moments App 中，我们为 Debug Target 使用了 Automatically manage signing 的方式来管理证书和 Provisioning Profile。这里请你思考一下，这样做有什么好处呢？</p>
</blockquote>
<p data-nodeid="1392">可以把你的答案写到留言区哦。下一讲我将介绍如何使用自动化构建来解决大量重复性工作的问题。</p>
<p data-nodeid="1393"><strong data-nodeid="1700">源码地址</strong></p>
<blockquote data-nodeid="1394">
<p data-nodeid="1395" class="">Fastfile 文件地址：<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/fastlane/Fastfile#L72-L207?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1704">https://github.com/lagoueduCol/iOS-linyongjian/blob/main/fastlane/Fastfile#L72-L207</a></p>
</blockquote>

---

### 精选评论

##### **逸：
> 额，猜测一下吧，开发环境开自动的好处就是不需要纠结证书和描述文件的选择，也不担心会选错描述问题件,总之能跑就行

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，如果有些开发者只开发代码，不负责打包和签名等操作，就可以省了执行 match 那一步了。

##### **成：
> 林老师这写的步骤也太详细了吧，连在GitHub上创建私有库都一步步的讲解，简直就是给小白准备的，是小白的福音啊不过像GitHub里建仓库这种操作，对于看进阶教程的iOSer来说，可能会显得有些太基础了

##### *蛋：
> 老师，bundle exec fastlane add_device 注册设备的完整格式是怎么样的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; bundle exec fastlane add_device <设备的 UDID>，UDID 可以在手机连上 Mac，然后打开 Finder 能找到。假如使用 Firebase App Distribution 服务，用户想下载 App 的时候，我们就能 Firebase 看到这个 UDID。

##### **泽：
> “会通过 Keychain Access 程序生成一个后缀为 .certSigningRequest 的 Certificate Signing Request 文件和私钥文件”老师，在私钥那部分， 通过keychain请求一个.certSigningRequest这个证书，这个就是说的私钥.p12文件吗？通常是给别的小伙伴从证书导出一个.p12文件使用

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，是 .p12文件文件，但是使用 match 就无限导入导出 .p12文件了，只需要一条命令就搞定。

