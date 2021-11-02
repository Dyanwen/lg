<p data-nodeid="234383" class="">这一讲我将带你学习如何为 Serverless 应用编写单元测试。</p>
<p data-nodeid="234384">单元测试是保证代码质量和应用稳定性的重要手段，但很多同学却不喜欢写单元测试，觉得又麻烦，又要花很多时间，其实这与没有掌握正确的方法有很大关系。除此之外，还有的同学不知道怎么写单元测试，尤其是怎么对 Serverless 应用编写单元测试，<strong data-nodeid="234492">而这是所有 Serverless 开发者面临的问题。</strong> 我们团队在使用 Serverless 的初期也踩过很多坑，总结起来主要有以下难点：</p>
<ul data-nodeid="234385">
<li data-nodeid="234386">
<p data-nodeid="234387">Serverless 架构是分布式的，组成 Serverless 应用的函数是单独运行的，这些函数集合到一起组成分布式架构，你需要对独立函数和分布式应用都进行测试；</p>
</li>
<li data-nodeid="234388">
<p data-nodeid="234389">Serverless 架构依赖很多云服务，比如各种 FaaS、BaaS 等，这些云服务很难在本地模拟；</p>
</li>
<li data-nodeid="234390">
<p data-nodeid="234391">Serverless 架构是事件驱动的，事件驱动这种异步工作模式也很难在本地模拟。</p>
</li>
</ul>
<p data-nodeid="234392"><strong data-nodeid="234500">那怎么解决这些问题呢？</strong> 这就是今天这一讲的重点了。总的来说，这一讲我会先带你了解 Serverelss 应用的单元测试准则，这些准则可以指导你编写出更易测试的代码，然后会带你编写实际的单元测试，并总结出单元测试的一些最佳实践，让你能够学以致用。</p>
<h3 data-nodeid="234393">Serverless 单元测试准则</h3>
<p data-nodeid="234394">著名的 Scrum 联盟创始人 Mike Cohn 在 2012 年提出了测试金字塔理论：</p>
<p data-nodeid="234634" class=""><img src="https://s0.lgstatic.com/i/image2/M01/05/4E/CgpVE1_-wzCALcR9AAR8TB-B350408.png" alt="Lark20210113-175325.png" data-nodeid="234638"></p>
<div data-nodeid="234635"><p style="text-align:center">测试金字塔</p></div>


<p data-nodeid="234397">如果你写过单元测试，测试金字塔对你来说肯定不陌生，测试可分为单元测试、服务测试和 UI 测试，金字塔越上层测试速度越慢，成本越高，所以你应该写更多的单元测试。</p>
<p data-nodeid="234398"><strong data-nodeid="234510">可是 Serverless 应用依赖很多云服务，函数参数也与触发器强相关，要怎么写代码才能更方便写单元测试呢？我实践总结出了几条测试准则，供你参考：</strong></p>
<ol data-nodeid="234399">
<li data-nodeid="234400">
<p data-nodeid="234401">将业务逻辑和依赖的云服务分开，保持业务代码独立，使其更易于扩展和测试；</p>
</li>
<li data-nodeid="234402">
<p data-nodeid="234403">对业务逻辑编写充分的单元测试，保证业务代码的正确性；</p>
</li>
<li data-nodeid="234404">
<p data-nodeid="234405">对业务代码和云服务编写集成测试，保证应用的正确性。</p>
</li>
</ol>
<p data-nodeid="234406">我来带你看一个实际的例子。<strong data-nodeid="234519">假设你要实现一个功能：保存用户信息，保存成功后并发送欢迎邮件。</strong> 最简单、最好实现、但不好测试的代码就是下面这样：</p>
<pre class="lang-javascript" data-nodeid="234407"><code data-language="javascript"><span class="hljs-comment">// handler.js</span>
<span class="hljs-keyword">const</span> db = <span class="hljs-built_in">require</span>(<span class="hljs-string">'db'</span>).connect();
<span class="hljs-keyword">const</span> mailer = <span class="hljs-built_in">require</span>(<span class="hljs-string">'mailer'</span>);
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function">(<span class="hljs-params">event, context, callback</span>) =&gt;</span> {
  <span class="hljs-keyword">const</span> user = {
    <span class="hljs-attr">email</span>: event.email,
    <span class="hljs-attr">created_at</span>: <span class="hljs-built_in">Date</span>.now(),
  };

  <span class="hljs-comment">// 将用户信息存入数据库</span>
  db.saveUser(user, <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">err, userId</span>) </span>{
    <span class="hljs-keyword">if</span> (err) {
      callback(<span class="hljs-string">'保存用户信息失败'</span>);
    } <span class="hljs-keyword">else</span> {
      <span class="hljs-comment">// 存入成功后，为用户发送一封邮件</span>
      <span class="hljs-keyword">const</span> success = mailer.sendWelcomeEmail(event.email);
      <span class="hljs-keyword">if</span> (success) {
        <span class="hljs-comment">// 如果发送邮件成功，则通过回调函数返回 userId</span>
        callback(<span class="hljs-literal">null</span>, userId);
      } <span class="hljs-keyword">else</span> {
        <span class="hljs-comment">// 如果发送邮件失败，则通过回调函数告诉调用方发送邮件失败</span>
        callback(<span class="hljs-string">`发送邮件（<span class="hljs-subst">${user.email}</span>）失败`</span>);
      }
    }
  });
};
</code></pre>
<p data-nodeid="234408">按照测试准则一，这份代码存在这样几个问题。</p>
<ul data-nodeid="234409">
<li data-nodeid="234410">
<p data-nodeid="234411"><strong data-nodeid="234525">业务逻辑没有和 FaaS 服务分开：</strong> 因为 handler 是 FaaS 的入口函数，handler 的参数是由具体 FaaS 平台实现的，比如函数计算、Lambda 等，不同 FaaS 平台实现不一样。</p>
</li>
<li data-nodeid="234412">
<p data-nodeid="234413"><strong data-nodeid="234530">单元测试依赖数据库（db）和邮件服务（mailer）：</strong> 这些服务都需要发送网络请求。</p>
</li>
</ul>
<p data-nodeid="234414">所以让我们将这段代码进行重构，使其更易于测试：</p>
<pre class="lang-javascript" data-nodeid="234415"><code data-language="javascript"><span class="hljs-comment">// src/users.js</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Users</span> </span>{
  <span class="hljs-keyword">constructor</span>(db, mailer) {
    <span class="hljs-keyword">this</span>.db = db;
    <span class="hljs-keyword">this</span>.mailer = mailer;
  }
  save(email, callback) {
    <span class="hljs-keyword">const</span> user = {
      <span class="hljs-attr">email</span>: email,
      <span class="hljs-attr">created_at</span>: <span class="hljs-built_in">Date</span>.now(),
    };
    <span class="hljs-comment">// 将用户信息存入数据库</span>
    <span class="hljs-keyword">this</span>.db.saveUser(user, (err, id) =&gt; {
      <span class="hljs-keyword">if</span> (err) {
        callback(<span class="hljs-string">'保存用户信息失败'</span>);
      } <span class="hljs-keyword">else</span> {
        <span class="hljs-comment">// 存入成功后，为用户发送一封邮件</span>
        <span class="hljs-keyword">const</span> success = <span class="hljs-keyword">this</span>.mailer.sendWelcomeEmail(email);
        <span class="hljs-keyword">if</span> (success) {
          <span class="hljs-comment">// 如果发送邮件成功，则通过回调函数返回 userId</span>
          callback(<span class="hljs-literal">null</span>, id);
        } <span class="hljs-keyword">else</span> {
          <span class="hljs-comment">// 如果发送邮件失败，则通过回调函数告诉调用方发送邮件失败</span>
          callback(<span class="hljs-string">`发送邮件（<span class="hljs-subst">${email}</span>）失败`</span>);
        }
      }
    });
  }
}
<span class="hljs-built_in">module</span>.exports = Users;
</code></pre>
<pre class="lang-javascript" data-nodeid="234416"><code data-language="javascript"><span class="hljs-comment">// handler.js</span>
<span class="hljs-keyword">const</span> db = <span class="hljs-built_in">require</span>(<span class="hljs-string">'db'</span>).connect();
<span class="hljs-keyword">const</span> mailer = <span class="hljs-built_in">require</span>(<span class="hljs-string">'mailer'</span>);
<span class="hljs-keyword">const</span> Users = <span class="hljs-built_in">require</span>(<span class="hljs-string">'./src/users'</span>);
<span class="hljs-comment">// 初始化 User 实例</span>
<span class="hljs-keyword">let</span> users = <span class="hljs-keyword">new</span> Users(db, mailer);
<span class="hljs-built_in">module</span>.exports.saveUser = <span class="hljs-function">(<span class="hljs-params">event, context, callback</span>) =&gt;</span> {
  users.save(event.email, callback);
};
</code></pre>
<p data-nodeid="234417">你可以看到，我们将存储数据和发送邮件的业务逻辑单独拆分到了 Users 类中，并且 User 类提供构造函数，注入 db 和 mailer 这两个依赖。db 和 mailer 的初始化逻辑和依赖 FaaS 的 handler 函数单独放在 handler.js 文件中，该文件中的代码修改频率更低。</p>
<p data-nodeid="234418">这样修改后代码就满足了准则一，我们的业务逻辑也完全不依赖任何外部服务了，在单元测试时既可以使用真实的 db 和 mailer 服务，也可以模拟 db 和 mailer 类，使单元测试更简单。并且你的代码也更易于扩展，当你想要将代码迁移到其他 FaaS 平台，你不用修改业务逻辑，只需要提供一个 handler.js 使其适用于新的 FaaS 平台，从而避免云厂商绑定。</p>
<p data-nodeid="234419">此外根据准则二，我们还需要对业务逻辑进行充分的单元测试，也就是需要对 User 类编写单元测试，具体怎么做呢？接下来就让我们进入单元测试实践部分。</p>
<h3 data-nodeid="234420">Serverless 单元测试实践</h3>
<p data-nodeid="234421">首先你需要选择一个测试框架，Node.js 中有很多优秀的测试框架，比如 Jest、Mocha 等，我比较喜欢的是 Jest，因为它可以零配置上手使用、内置 Mock 功能、提供了完整的测试覆盖率报告等。</p>
<p data-nodeid="234422">你可以使用 npm install -D jest 来安装。然后在 package.json 中添加一个 jest 的命令即可。</p>
<p data-nodeid="234423">为了方便管理所有测试用例，你可以创建一个 <code data-backticks="1" data-nodeid="234539">__test__</code>目录，然后在里面新建名为 <code data-backticks="1" data-nodeid="234541">users.test.js</code>的文件编写 Users 类的测试。Jest 默认会将<code data-backticks="1" data-nodeid="234543">__test__</code>目录或包含 test 关键字文件中的代码当作单元测试。</p>
<p data-nodeid="234424">为了方便你进行实践，我提供了一个份<a href="https://github.com/nodejh/serverless-class/tree/master/08" data-nodeid="234548">示例代码</a>供你参考，你也可以通过 Git 命令下载使用：</p>
<pre class="lang-java" data-nodeid="234425"><code data-language="java">$ git clone git<span class="hljs-meta">@github</span>.com:nodejh/serverless-class.git
$ cd <span class="hljs-number">08</span>/unit-testing/
</code></pre>
<p data-nodeid="234426">完整的目录结构如下：</p>
<pre class="lang-shell" data-nodeid="234427"><code data-language="shell">.
├── README.md
├── __tests__
│   └── users.test.js
├── src
│   └── users.js
├── handler.js
├── node_modules
├── package.json
└── serverless.yaml
</code></pre>
<p data-nodeid="234428">为了保证业务逻辑的正确性，我们就需要对 Users 类进行测试。Users 类主要提供了 save 方法，save 方法的功能是把用户信息存入数据库，然后给用户发送一封邮件。代码运行中，可能存在几种情况：</p>
<ul data-nodeid="234429">
<li data-nodeid="234430">
<p data-nodeid="234431">用户信息写入数据库成功，发送邮件成功；</p>
</li>
<li data-nodeid="234432">
<p data-nodeid="234433">用户信息写入数据库成功，发送邮件失败；</p>
</li>
<li data-nodeid="234434">
<p data-nodeid="234435">用户信息写入数据库失败。</p>
</li>
</ul>
<p data-nodeid="234436"><strong data-nodeid="234559">这个时候你可能会感觉有点困难了，</strong> 写数据库或发邮件都依赖远程服务，并且还有那么多异常情况要考虑，怎么进行测试呢？对于 save 方法来说，它本质上不需要关心远程服务，只需考虑分支逻辑的正确性，所以这个地方我们可以对 db 和 mailer 进行模拟，模拟 db 和 mailer 的各自异常情况，然后观察 save 方法的执行结果是否正确。</p>
<p data-nodeid="234437">Jest 提供了 Mock 功能，可以让我们对类或函数进行模拟。由于 save 方法主要使用到了 db.saveUser 和 mailer.sendWelcomeEmail 这两个函数，所以我们只需要对这两个函数进行模拟：</p>
<pre class="lang-javascript" data-nodeid="234438"><code data-language="javascript"> <span class="hljs-keyword">const</span> db = {
    <span class="hljs-attr">saveUser</span>: jest.fn(<span class="hljs-function">(<span class="hljs-params">user, callback</span>) =&gt;</span> callback(<span class="hljs-literal">null</span>, <span class="hljs-number">1</span>)),
  };
  <span class="hljs-keyword">const</span> mailer = {
    <span class="hljs-attr">sendWelcomeEmail</span>: jest.fn(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-literal">true</span>),
  };
</code></pre>
<p data-nodeid="234439">在上面的代码中，db.saveUser 的参数是 user 和回调函数 callback，当函数执行完毕会调用回调函数，回调函数的参数分别是 null 和 1，表示 db.saveUser 执行成功。mailerMock.sendWelcomeEmail 的返回值则为 true，表示发送邮件成功。</p>
<p data-nodeid="234440">接下来就可以针对 save 方法编写第一个测试用例：</p>
<pre class="lang-javascript" data-nodeid="234441"><code data-language="javascript"><span class="hljs-keyword">const</span> Users = <span class="hljs-built_in">require</span>(<span class="hljs-string">'../src/users'</span>);
test(<span class="hljs-string">'用户信息写入数据库成功，发送邮件成功'</span>, () =&gt; {
  <span class="hljs-comment">// 模拟 db.saveUser，并调用成功</span>
  <span class="hljs-keyword">const</span> db = {
    <span class="hljs-attr">saveUser</span>: jest.fn(<span class="hljs-function">(<span class="hljs-params">user, cb</span>) =&gt;</span> cb(<span class="hljs-literal">null</span>, <span class="hljs-number">1</span>)),
  };
  <span class="hljs-comment">// 模拟 mailer.sendWelcomeEmail，并调用成功</span>
  <span class="hljs-keyword">const</span> mailer = {
    <span class="hljs-attr">sendWelcomeEmail</span>: jest.fn(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-literal">true</span>),
  };

  <span class="hljs-keyword">const</span> users = <span class="hljs-keyword">new</span> Users(db, mailer);
  <span class="hljs-keyword">const</span> email = <span class="hljs-string">'test@gmail.com'</span>;
  users.save(email, (err, userId) =&gt; {
    <span class="hljs-comment">// 第一个断言，保存用户信息后的结果为 null</span>
    expect(err).toBeNull();
    <span class="hljs-comment">// 第二个断言，保存并发送</span>
    expect(userId).toBe(<span class="hljs-number">1</span>);
  });
});
</code></pre>
<p data-nodeid="234442">首先我们模拟了 db 和 mailer 这两个远程服务，并假设其运行结果都是正常的，然后用模拟的 db 和 mailer 来初始化 users 类，接下来调用 users.save 方法。当 db 和 mailer 都执行正常后，users.save 的回调函数参数就分别应该是模拟的 null 和 1，所以我们在 users.save 的回调函数中添加了 expect(err).toBeNull() 和 expect(userId).toBe(1) 这两个断言。</p>
<p data-nodeid="234443">使用 npm run test 测试一下，运行结果如下：</p>
<pre class="lang-shell" data-nodeid="234444"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> npm run <span class="hljs-built_in">test</span></span>
<span class="hljs-meta">&gt;</span><span class="bash"> unit-testing@1.0.0 <span class="hljs-built_in">test</span></span>
<span class="hljs-meta">&gt;</span><span class="bash"> jest</span>
 PASS  __test__/users.test.js
  ✓ 用户信息写入数据库成功，发送邮件成功 (11 ms)
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        3.391 s
Ran all test suites.
</code></pre>
<p data-nodeid="234445">类似地，我们还需要增加其他两种情况的测试用例：</p>
<pre class="lang-javascript" data-nodeid="234446"><code data-language="javascript">test(<span class="hljs-string">'用户信息写入数据库成功，发送邮件失败'</span>, () =&gt; {
  <span class="hljs-keyword">const</span> db = {
    <span class="hljs-attr">saveUser</span>: jest.fn(<span class="hljs-function">(<span class="hljs-params">user, cb</span>) =&gt;</span> cb(<span class="hljs-literal">null</span>, <span class="hljs-number">1</span>)),
  };
  <span class="hljs-keyword">const</span> mailer = {
    <span class="hljs-attr">sendWelcomeEmail</span>: jest.fn(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-literal">false</span>),
  };

  <span class="hljs-keyword">const</span> users = <span class="hljs-keyword">new</span> Users(db, mailer);
  <span class="hljs-keyword">const</span> email = <span class="hljs-string">'test@gmail.com'</span>;
  users.save(email, (err, userId) =&gt; {
    expect(err).toBe(<span class="hljs-string">`发送邮件（<span class="hljs-subst">${email}</span>）失败`</span>);
    expect(userId).toBeUndefined();
  });
});

test(<span class="hljs-string">'用户信息写入数据失败'</span>, () =&gt; {
  <span class="hljs-keyword">const</span> db = {
    <span class="hljs-attr">saveUser</span>: jest.fn(<span class="hljs-function">(<span class="hljs-params">user, cb</span>) =&gt;</span> cb(<span class="hljs-keyword">new</span> <span class="hljs-built_in">Error</span>(<span class="hljs-string">'Internal Error'</span>))),
  };
  <span class="hljs-keyword">const</span> mailer = {
    <span class="hljs-attr">sendWelcomeEmail</span>: jest.fn(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-literal">false</span>),
  };

  <span class="hljs-keyword">const</span> users = <span class="hljs-keyword">new</span> Users(db, mailer);
  <span class="hljs-keyword">const</span> email = <span class="hljs-string">'test@gmail.com'</span>;
  users.save(email, (err, userId) =&gt; {
    expect(err).toBe(<span class="hljs-string">'保存用户信息失败'</span>);
    expect(userId).toBeUndefined();
  });
});
</code></pre>
<p data-nodeid="234447">至此，关于 Users 类的完整单元测试就写好了。我在示例代码中还添加了<code data-backticks="1" data-nodeid="234567">test:coverage</code>这个命令用来运行单元测试并生成测试覆盖率：</p>
<pre class="lang-java" data-nodeid="234448"><code data-language="java">$ npm run test:coverage
&gt; unit-testing@1.0.0 test:coverage
&gt; jest --collect-coverage
 PASS  __test__/users.test.js
  ✓ 用户信息写入数据库成功，发送邮件成功 (7 ms)
  ✓ 用户信息写入数据库成功，发送邮件失败
  ✓ 用户信息写入数据失败 (1 ms)
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
----------|---------|----------|---------|---------|-------------------
All files |     100 |      100 |     100 |     100 |
 users.js |     100 |      100 |     100 |     100 |
----------|---------|----------|---------|---------|-------------------
Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        2.983 s
Ran all test suites.
</code></pre>
<p data-nodeid="234449">从运行结果中可以看的 user.js 的覆盖率是 100%，说明所有代码都经过了测试。100% 的单测覆盖率也应该是每个追求极致的程序员的目标。</p>
<p data-nodeid="234450">有了完整的单元测试，我们就再也不用担心修改代码引入 Bug 了。假如某天你修改了业务逻辑，比如回调函数不再返回 userId 了，类似下面这样：</p>
<pre class="lang-javascript" data-nodeid="234451"><code data-language="javascript">  save(email, callback) {
    <span class="hljs-comment">// ...</span>
    <span class="hljs-keyword">this</span>.db.saveUser(user, (err, userId) =&gt; {
      <span class="hljs-keyword">if</span> (err) { 
        callback(<span class="hljs-string">'保存用户信息失败'</span>);
      } <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">const</span> success = <span class="hljs-keyword">this</span>.mailer.sendWelcomeEmail(email);
        <span class="hljs-keyword">if</span> (success) {
          <span class="hljs-comment">// 发送邮件成功后，不再返回 userId</span>
          callback(<span class="hljs-literal">null</span>);
        } <span class="hljs-keyword">else</span> {
          callback(<span class="hljs-string">`发送邮件（<span class="hljs-subst">${email}</span>）失败`</span>);
        }
      }
    });
  }
</code></pre>
<p data-nodeid="234452">再去运行单元测试，单元测试就无法通过了：</p>
<pre class="lang-shell" data-nodeid="234453"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> npm run <span class="hljs-built_in">test</span></span>
<span class="hljs-meta">&gt;</span><span class="bash"> unit-testing@1.0.0 <span class="hljs-built_in">test</span></span>
<span class="hljs-meta">&gt;</span><span class="bash"> jest</span>
 FAIL  __test__/users.test.js
  ✕ 用户信息写入数据库成功，发送邮件成功 (14 ms)
  ✓ 用户信息写入数据库成功，发送邮件失败 (1 ms)
  ✓ 用户信息写入数据失败
  ● 用户信息写入数据库成功，发送邮件成功
    expect(received).toBe(expected) // Object.is equality
    Expected: 1
    Received: undefined
      18 |     expect(err).toBeNull();
      19 |     // 第二个断言，保存并发送
    &gt; 20 |     expect(userId).toBe(1);
         |                    ^
      21 |   });
      22 | });
      23 |
      at callback (__test__/users.test.js:20:20)
      at cb (src/users.js:19:11)
      at Object.&lt;anonymous&gt; (__test__/users.test.js:7:37)
      at Users.save (src/users.js:13:13)
      at Object.&lt;anonymous&gt; (__test__/users.test.js:16:9)
Test Suites: 1 failed, 1 total
Tests:       1 failed, 2 passed, 3 total
Snapshots:   0 total
Time:        3.054 s
Ran all test suites.
</code></pre>
<p data-nodeid="234454">从单元测试结果就可以看出业务逻辑发生了不兼容变更，这时就需要考虑该变更对上下游的影响，避免造成线上业务风险，在确认没有风险后再修改测试用例。</p>
<p data-nodeid="234455">在业务变得越来越复杂的时候，你的代码也会越来越多，单元测试也会越来越多。<strong data-nodeid="234578">为了更好地管理代码，我建议单元测试的目录结构和业务代码结构保持一致，</strong> 比如未来你的代码可能演化为 MVC（Modle-Controller-View）三层架构，则代码目录结构应该是下面这样：</p>
<pre class="lang-java" data-nodeid="234456"><code data-language="java">.
├── README.md
├── __tests__
│   └── controllers
│   |   └── users.test.js
│   └── models
│   |   └── users.test.js
│   └── views
├── src
│   └── controllers
│   |   └── users.test.js
│   └── models
│   |   └── users.test.js
│   └── views
├── handler.js
├── node_modules
├── <span class="hljs-keyword">package</span>-lock.json
├── <span class="hljs-keyword">package</span>.json
└── serverless.yml
</code></pre>
<p data-nodeid="234457">其中 src 目录中是业务源代码， <code data-backticks="1" data-nodeid="234580">__test__</code>中是单元测试代码， 两个目录结构完全相同，这样代码的结构层次就非常清晰。</p>
<p data-nodeid="234458">讲到这儿，我想你应该对开篇提到的 Serverless 单元测试的难点有了理解了吧？<strong data-nodeid="234586">要解决这些难点，主要就是要将业务代码和依赖的云服务分离开来，这样才能方便测试。</strong></p>
<p data-nodeid="234459">当然了，我在本讲开篇也提到了，虽然单元测试很好，但也有很多人不喜欢写单元测试，主要是觉得麻烦。所以为了让单元测试能够为开发提升价值，而不带来负担，<strong data-nodeid="234591">我总结了一些单元测试的最佳实践，希望能够给你一些帮助。</strong></p>
<ul data-nodeid="234460">
<li data-nodeid="234461">
<p data-nodeid="234462"><strong data-nodeid="234596">速度</strong>：单元测试的速度要足够快，因为单元测试运行非常频繁，是用来辅助开发的，如果运行速度慢，则会影响开发效率，我们团队要求单个测试小于200ms，整个系统的测试小于10分钟。</p>
</li>
<li data-nodeid="234463">
<p data-nodeid="234464"><strong data-nodeid="234601">隔离外部调用</strong>：单元测试需要隔离一切外部调用，比如不能使用其他真实类、不能读磁盘、不能有网络调用、不能写数据库、不能依赖环境变量、不能依赖系统时间等。</p>
</li>
<li data-nodeid="234465">
<p data-nodeid="234466"><strong data-nodeid="234606">模拟</strong>：必要时需要对外部进行模拟，以确保单元测试不被外部环境所影响。由于模拟外部 API 可能会导致代码内部行为发送改变，所以要注意按照最新外部 API 描述进行模拟。</p>
</li>
<li data-nodeid="234467">
<p data-nodeid="234468"><strong data-nodeid="234611">单一职责</strong>：一个测试用例只用于验证一个行为;</p>
</li>
<li data-nodeid="234469">
<p data-nodeid="234470"><strong data-nodeid="234616">自描述</strong>：单元测试是代码最好的文档，也是方法最好的描述，因此单元测试需要能够明确代码的意图。</p>
</li>
</ul>
<p data-nodeid="234471">最后，单元测试并不是测试的全部，单元测试只是用来保证单个功能、组件的正确性。在 Serverless 中，你依旧需要使用集成测试来验证所有组件集成到一起时运行是否正常，所以我也建议你对 Serverless 应用进行集成测试。</p>
<h3 data-nodeid="234472">总结</h3>
<p data-nodeid="234473">单元测试一直是困扰 Serverless 开发者的一大难题。在本节课中，我首先展示了一段难以编写单元测试的代码示例，然后讨论了为什么 Serverless 应用编写单元测试难、应该如何编写易测试的代码，以及如何编写单元测试，最后介绍了我在编写单元测试过程中的一些最佳实践。<strong data-nodeid="234623">关于这一讲，我想强调这几个点：</strong></p>
<ul data-nodeid="234474">
<li data-nodeid="234475">
<p data-nodeid="234476">Serverless 应用由于其分布式、依赖云服务、事件驱动等特性，导致编写单元测试很困难；</p>
</li>
<li data-nodeid="234477">
<p data-nodeid="234478">为了方便编写单元测试，需要将业务逻辑和依赖的云服务分离开来；</p>
</li>
<li data-nodeid="234479">
<p data-nodeid="234480">编写单元测试时，需要考虑速度、隔离性、单一职责等因素，避免单元测试成为开发的负担；</p>
</li>
<li data-nodeid="234481">
<p data-nodeid="234482">好的单元测试应该是自描述的，能对代码进行解释说明。</p>
</li>
</ul>
<p data-nodeid="235651">总的来说，对 Serverless 应用编写单元测试的前提是将业务代码和云服务依赖分离，在设计和编写业务代码时就需要考虑代码是否利于测试，在此基础上，业务代码的单元测试和传统应用单元测试的方法是互通的。但单元测试只是保证系统质量的一部分，你依旧需要编写集成测试，来保证整个系统的质量和稳定性。</p>
<p data-nodeid="235652" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/05/4E/CgpVE1_-w06AWX__AAEYrgALpK0776.png" alt="Lark20210113-175329.png" data-nodeid="235656"></p>

<p data-nodeid="235397">最后我给你留的作业：亲自实践一下 Serverless 应用的单元测试和集成测试。</p>




<p data-nodeid="234485" class="">示例代码地址：<a href="https://github.com/nodejh/serverless-class/tree/master/08/unit-testing" data-nodeid="234633">https://github.com/nodejh/serverless-class/tree/master/08/unit-testing</a></p>

---

### 精选评论


