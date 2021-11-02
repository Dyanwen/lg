<p data-nodeid="3696" class="">从这一讲开始，我将为你介绍如何进行性能测试。</p>
<p data-nodeid="3697">通过代码实战部分你可以看到，我在写每个功能的时候，都会编写测试代码。那是因为 TDD（Test-Driven Development，测试驱动开发）中提倡先编写测试代码，然后再编写功能代码，每做一个修改后，都要执行一次单元测试和基准测试，以此来验证功能和性能是否有问题。</p>
<p data-nodeid="3698">特别是业务系统代码经常变更，单元测试和基准测试也就显得非常重要。而每种语言都有自己的测试框架，比如 Go 语言，它是门注重工程效率的语言，有着非常强大的工具链，它自带的测试框架，能满足我们大部分测试要求。</p>
<p data-nodeid="3699">所以，这一讲，我将详细介绍如何使用 Go 测试框架做性能测试中的单元测试和基准测试。</p>
<h3 data-nodeid="3700">单元测试</h3>
<p data-nodeid="3701">Go 测试框架中支持白盒测试和黑盒测试。现在我就以 infrastructure/stores/cache.go 这个文件为例，给你详细介绍下如何做单元测试。</p>
<h4 data-nodeid="3702">总体步骤</h4>
<p data-nodeid="3703">总的来说，用 Go 测试框架做单元测试主要有这几个步骤。</p>
<p data-nodeid="3704">第一，Go 测试框架要求测试代码文件名以 _test.go 结尾。为了测试 cache.go，我们需要在 infrastructure/stores 目录下创建一个 cache_test.go 文件。</p>
<p data-nodeid="3705">第二，cache_test.go 中第一行如果是 package stores，则表示该测试是白盒测试，这意味着除了这个包的全局函数外，你还可以测试它的私有函数；如果是 package stores_test，则表示黑盒测试，你只可以测试全局函数，里面的具体实现对于你来说是个黑盒子。</p>
<p data-nodeid="3706">第三，Go 测试框架要求单元测试函数需要以 Test 开头。为了测试 IntCache 和 ObjCache，我们需要实现 TestIntCache 和 TestObjCache 这两个函数，它们的参数类型都是 testing.T 指针。</p>
<p data-nodeid="3707">第四，在测试过程中，如果发现错误，可以通过测试框架的 Error 方法或者 Fatal 方法输出错误。不同的是，Error 方法仅仅输出错误，而 Fatal 方法却会结束当前测试。</p>
<p data-nodeid="3708">第五，在终端进入项目根目录下，执行 go test ./infrastructure/stores 命令，将会执行 infrastructure/stores 目录下的所有单元测试。</p>
<p data-nodeid="3709">结果通常会有三列：</p>
<ul data-nodeid="3710">
<li data-nodeid="3711">
<p data-nodeid="3712">第一列是测试结果，ok 表示成功，FAIL 表示失败；</p>
</li>
<li data-nodeid="3713">
<p data-nodeid="3714">第二列是被测试包的完整路径；</p>
</li>
<li data-nodeid="3715">
<p data-nodeid="3716">第三列是执行测试耗费的时间。</p>
</li>
</ul>
<p data-nodeid="3717">当测试失败时，输出结果还会告诉你在哪一行报错了，格式通常是：第一行是测试函数，第二行是文件名和该文件的第几行，后面再跟上具体错误日志。比如，TestIntCache 这个测试函数在 cache_test.go 文件第 19 行报错。如下图所示：</p>
<p data-nodeid="3718"><img src="https://s0.lgstatic.com/i/image6/M00/04/84/Cgp9HWAszXyAWFS6AADUOlqkKB0107.png" alt="Drawing 0.png" data-nodeid="3793"></p>
<h4 data-nodeid="3719">覆盖率</h4>
<p data-nodeid="3720">在单元测试中，除了测试功能是否正常外，还有个很重要的指标：<strong data-nodeid="3803">覆盖率</strong>。覆盖率用来衡量单元测试覆盖了多少代码，它是由覆盖测试统计出来的。<strong data-nodeid="3804">覆盖率越高，说明单元测试越完备，对代码质量更有保障。</strong></p>
<p data-nodeid="3721">Go 的覆盖测试使用很简单，在 go test 命令后加上 -cover 参数即可。如执行 go test -cover ./infrastructure/stores 将会在之前的输出结果后面加上 coverage 开头的日志。比如 coverage: 61.0% of statements 。</p>
<p data-nodeid="3722">虽然覆盖测试工具用起来很简单，但要想将覆盖率提升到 100% 则是非常困难。</p>
<p data-nodeid="3723">第一，你需要找出来哪些地方没有覆盖到。</p>
<p data-nodeid="3724">这里我们需要用到 -coverprofile 参数，将详细的结果输出到文件中。比如 go test -coverprofile cover.out 便是将覆盖测试的结果输出到 cover.out 中。然后我们用命令 go tool cover -html=cover.out -o cover.html 将测试结果输出为 html 文件，再用浏览器打开便可以看到哪些代码被单元测试覆盖到了，哪些没有被覆盖到。其中绿色部分表示已覆盖到的代码，红色部分表示没有覆盖到。效果如下：</p>
<p data-nodeid="3725"><img src="https://s0.lgstatic.com/i/image6/M00/04/84/Cgp9HWAszYyAF5LnAAE2Ig80LGU118.png" alt="Drawing 1.png" data-nodeid="3811"></p>
<p data-nodeid="3726">我们可以看到，Add 方法有大片代码没被测试覆盖到，我们可以通过修改测试代码来覆盖这部分代码的测试。</p>
<p data-nodeid="3727">第二，你需要对代码逻辑非常熟悉，特别是熟悉代码中的各种边界条件，并在测试代码中构造出这些边界条件来测试。</p>
<p data-nodeid="3728">假如代码里有三个条件分支 A、B、C，你需要将这三个条件都构造出来，才能覆盖到它们各自分支下面的代码。这意味着，你在编写测试代码的时候，需要编写大量的测试用例，也就是构造大量的能触发边界条件的参数。如果没有好的代码功底，你的测试代码容易因为大量参数而变得混乱、臃肿。</p>
<p data-nodeid="3729">有时候，一个功能可能会有多个参数，每个参数又有多个边界值。当我们采用控制变量法来测试的时候，所有参数的边界值又会产生多种组合，这个数量通常是每个参数边界值数量的乘积。比如 A 参数有 2 个边界值，B 参数有 3 个边界值，那么将会存在 6 种组合 。为了便于管理，我们需要将这些参数提取到核心测试代码之外，用数组和循环来管理测试用例。</p>
<p data-nodeid="3730">比如我们在测试 IntCache 的 Add 方法时，按照 key、delta 这两个参数的边界情况，就有 key 存在、不存在两种情况与 delta 为 -1、0、1 这三种情况的组合。我们将 Add 方法的 key、delta 参数以及期望返回值的配置，用一个结构体数组来管理，并用循环遍历数组，根据数组中每个元素的值调用 Add 方法并判断返回值与期望值是否相等。如果不相等则报错，输出有问题的测试用例，并终止测试。我们可以得到如下代码：</p>
<pre class="lang-go" data-nodeid="3731"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">TestIntCache_Add</span><span class="hljs-params">(t *testing.T)</span></span> {
   cache := NewIntCache()
   cases := []<span class="hljs-keyword">struct</span> {
      key    <span class="hljs-keyword">string</span>
      delta  <span class="hljs-keyword">int64</span>
      expect <span class="hljs-keyword">int64</span>
   }{
      {<span class="hljs-string">"test1"</span>, <span class="hljs-number">0</span>, <span class="hljs-number">0</span>},
      {<span class="hljs-string">"test1"</span>, <span class="hljs-number">1</span>, <span class="hljs-number">1</span>},
      {<span class="hljs-string">"test1"</span>, <span class="hljs-number">-1</span>, <span class="hljs-number">0</span>},
      {<span class="hljs-string">"test1"</span>, <span class="hljs-number">0</span>, <span class="hljs-number">0</span>},
      {<span class="hljs-string">"test2"</span>, <span class="hljs-number">1</span>, <span class="hljs-number">1</span>},
      {<span class="hljs-string">"test3"</span>, <span class="hljs-number">-1</span>, <span class="hljs-number">-1</span>},
   }
   <span class="hljs-keyword">for</span> _, c := <span class="hljs-keyword">range</span> cases {
      <span class="hljs-keyword">if</span> cache.Add(c.key, c.delta) != c.expect {
         t.Fatal(c)
      }
   }
}
</code></pre>
<p data-nodeid="3732">再次执行覆盖测试，你将看到代码覆盖率从 61.0% 提升到了 76.3%，Add 方法只剩下一行代码没覆盖到，而这行代码需要通过构造并发条件才能覆盖到。效果如下：</p>
<p data-nodeid="3733"><img src="https://s0.lgstatic.com/i/image6/M00/04/81/CioPOWAszaKAUq7-AAFghWXSHqs282.png" alt="Drawing 2.png" data-nodeid="3820"></p>
<h3 data-nodeid="3734">基准测试</h3>
<p data-nodeid="3735">基准测试属于性能测试，通常用于对具体的功能函数做性能分析，比如加密算法函数。基准测试需要有对比测试，以便衡量不同代码实现之间的性能差异，从中选取性能最好的实现方式。</p>
<p data-nodeid="3736">在 Go 测试框架中，基准测试函数需要以 Benchmark 开头。比如 BenchmarkIntCache_Set 表示对 IntCache 的 Set 方法进行基准测试。为了与 sync.Map 和 ObjCache 对比，我们还需要实现 BenchmarkObjCache_Set 和 BenchmarkSyncMap_Set 这两个函数。</p>
<p data-nodeid="3737">不过，当我们要测试对象的方法比较多时，为每个对象的每个方法都实现一个独立的测试函数并不是很方便。对此，我们通常我们采用分组测试的方式。Go 测试框架提供了一个 Run 方法可用于执行分组中的子测试，它有两个参数，第一个是子测试的名称，第二个是测试函数。</p>
<p data-nodeid="3738">这里，我们可以实现一个 BenchmarkCache_Set 函数来作为 Set 方法的测试组入口，里面包含 intCache、objCache、syncMap 这三个子测试。这样我们可以为每个子测试统一初始化公共资源，并复用核心代码逻辑。</p>
<p data-nodeid="3739">为了复用代码，我实现了一个 benchmarkCacheSet 函数。它有三个参数：</p>
<ol data-nodeid="3740">
<li data-nodeid="3741">
<p data-nodeid="3742">测试框架生成的 testing.B 对象指针；</p>
</li>
<li data-nodeid="3743">
<p data-nodeid="3744">setter 函数，它有 key 和 val 这两个参数，帮助设置被测对象中的 KV 值；</p>
</li>
<li data-nodeid="3745">
<p data-nodeid="3746">字符串数组 keys，表示用于测试的 key 集合。</p>
</li>
</ol>
<p data-nodeid="3747">在函数中，我们调用框架的 ReportAllocs 方法，用于输出内存分配信息。在循环开始前，调用框架的 StartTimer 开始计时，循环结束后调用 StopTimer 结束计时。</p>
<p data-nodeid="3748">在 BenchmarkCache_Set 中先初始化 keys，然后在三个子测试中分别初始化 IntCache、ObjCache、sync.Map，生成各自的 setter，并调用 benchmarkCacheSet 函数。</p>
<p data-nodeid="3749">最终，我们的测试代码如下：</p>
<pre class="lang-go" data-nodeid="3750"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">benchmarkCacheSet</span><span class="hljs-params">(b *testing.B, setter <span class="hljs-keyword">func</span>(key <span class="hljs-keyword">string</span>, val <span class="hljs-keyword">int64</span>)</span>, <span class="hljs-title">keys</span> []<span class="hljs-title">string</span>)</span> {
   b.ReportAllocs()
   b.StartTimer()
   l := <span class="hljs-built_in">len</span>(keys)
   <span class="hljs-keyword">for</span> i := <span class="hljs-number">0</span>; i &lt; b.N; i++ {
      setter(keys[i%l], <span class="hljs-keyword">int64</span>(i))
   }
   b.StopTimer()
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">BenchmarkCache_Set</span><span class="hljs-params">(b *testing.B)</span></span> {
   keys := <span class="hljs-built_in">make</span>([]<span class="hljs-keyword">string</span>, b.N, b.N)
   <span class="hljs-keyword">for</span> i := <span class="hljs-number">0</span>; i &lt; b.N; i++ {
      keys[i] = strconv.Itoa(i)
   }
   b.ResetTimer()
   b.Run(<span class="hljs-string">"intCache"</span>, <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(b *testing.B)</span></span> {
      c := NewIntCache()
      setter := <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(key <span class="hljs-keyword">string</span>, val <span class="hljs-keyword">int64</span>)</span></span> {
         c.Set(key, val)
      }
      benchmarkCacheSet(b, setter, keys)
   })
   b.Run(<span class="hljs-string">"objCache"</span>, <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(b *testing.B)</span></span> {
      c := NewObjCache()
      setter := <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(key <span class="hljs-keyword">string</span>, val <span class="hljs-keyword">int64</span>)</span></span> {
         c.Set(key, val)
      }
      benchmarkCacheSet(b, setter, keys)
   })
   b.Run(<span class="hljs-string">"syncMap"</span>, <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(b *testing.B)</span></span> {
      c := sync.Map{}
      setter := <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(key <span class="hljs-keyword">string</span>, val <span class="hljs-keyword">int64</span>)</span></span> {
         c.Store(key, val)
      }
      benchmarkCacheSet(b, setter, keys)
   })
}
</code></pre>
<p data-nodeid="3751">我们可以在终端执行命令 go test -bench=BenchmarkCache_Set ./infrastructure/stores 来运行上面实现的测试代码，你可以看到以 BenchmarkCache_Set 开头的三个子测试的结果。效果如下：</p>
<p data-nodeid="3752"><img src="https://s0.lgstatic.com/i/image6/M00/04/81/CioPOWAszayAVcGuAAF4gHUlcfM727.png" alt="Drawing 3.png" data-nodeid="3850"></p>
<p data-nodeid="3753">需要注意的是，每个子测试后面都有一个 -8，表示使用了 8 个 CPU 核。</p>
<p data-nodeid="3754">另外，Go 测试框架默认会使用所有的 CPU 核，但由于电脑上通常会运行其他的程序，使用所有的核可能会导致不同程序之间抢夺资源，影响测试结果。基于这一点，我们可以在参数中指定 CPU 核数来测试。</p>
<p data-nodeid="3755">具体来说，我们可以通过 -cpu 参数指定多种 CPU 核数来测试性能。比如，执行命令 go test -bench=BenchmarkCache_Set -cpu 1,2,3 ./infrastructure/stores 后，我们将看到每个子测试都用了不同 CPU 核数来测试。不过，由于我们的测试代码并没有用到并发，因此测试结果不受 CPU 核数的影响。效果如下：</p>
<p data-nodeid="3756"><img src="https://s0.lgstatic.com/i/image6/M00/04/81/CioPOWAszbOALrQcAAKC46smDD4638.png" alt="Drawing 4.png" data-nodeid="3858"></p>
<p data-nodeid="3757"><img src="https://s0.lgstatic.com/i/image6/M01/04/9B/Cgp9HWAtzUWADz17AAlrbAgwjRA894.png" alt="图片1.png" data-nodeid="3861"></p>
<h3 data-nodeid="3758">小结</h3>
<p data-nodeid="3759">这一讲我主要介绍了如何给 Go 程序做单元测试和基准测试，重点介绍了其中的一些小技巧。比如通过生成匿名函数的方式为不同的对象实现可复用的核心测试代码，通过数组和循环的方式管理测试用例。希望你已经掌握并能运用到工作中。</p>
<p data-nodeid="3760">接下来给你出个思考题：如何构造并发测试来提升 IntCache 的覆盖率？期待你在留言区讨论。</p>
<p data-nodeid="3761">这一讲就到这里了，下一讲我将给你介绍“如何使用 ab 命令和 pprof 分析性能”。到时见！</p>
<p data-nodeid="3872" class="te-preview-highlight">源码地址：<br>
<a href="https://github.com/lagoueduCol/MiaoSha-Yiletian/blob/main/infrastructure/stores/cache_test.go" data-nodeid="3879">https://github.com/lagoueduCol/MiaoSha-Yiletian/blob/main/infrastructure/stores/cache_test.go</a></p>

---

### 精选评论


