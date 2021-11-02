<p data-nodeid="88798" class="">gRPC proxy 是在 gRPC 层（L7）运行的无状态 etcd 反向代理，旨在<strong data-nodeid="88906">减少核心 etcd 集群上的总处理负载</strong>。gRPC proxy 合并了监视和 Lease API 请求，实现了水平可伸缩性。同时，为了保护集群免受滥用客户端的侵害，gRPC proxy 实现了键值对的读请求缓存。</p>
<p data-nodeid="88799">下面我们将围绕 gRPC proxy 基本应用、客户端端点同步、可伸缩的 API、命名空间的实现和其他扩展功能展开介绍。</p>
<h3 data-nodeid="88800">gRPC proxy 基本应用</h3>
<p data-nodeid="88801">首先我们来配置 etcd 集群，集群中拥有如下的静态成员信息：</p>
<p data-nodeid="89071" class=""><img src="https://s0.lgstatic.com/i/image6/M00/02/39/CioPOWAdDsCAOeHvAACGoxT-wS8912.png" alt="202125-92345.png" data-nodeid="89074"></p>

<p data-nodeid="88826">使用<code data-backticks="1" data-nodeid="88932">etcd grpc-proxy start</code>的命令开启 etcd 的 gRPC proxy 模式，包含上表中的静态成员：</p>
<pre class="lang-java" data-nodeid="88827"><code data-language="java">$ etcd grpc-proxy start --endpoints=http:<span class="hljs-comment">//192.168.10.7:2379,http://192.168.10.8:2379,http://192.168.10.9:2379 --listen-addr=192.168.10.7:12379</span>
{<span class="hljs-string">"level"</span>:<span class="hljs-string">"info"</span>,<span class="hljs-string">"ts"</span>:<span class="hljs-string">"2020-12-13T01:41:57.561+0800"</span>,<span class="hljs-string">"caller"</span>:<span class="hljs-string">"etcdmain/grpc_proxy.go:320"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"listening for gRPC proxy client requests"</span>,<span class="hljs-string">"address"</span>:<span class="hljs-string">"192.168.10.7:12379"</span>}
{<span class="hljs-string">"level"</span>:<span class="hljs-string">"info"</span>,<span class="hljs-string">"ts"</span>:<span class="hljs-string">"2020-12-13T01:41:57.561+0800"</span>,<span class="hljs-string">"caller"</span>:<span class="hljs-string">"etcdmain/grpc_proxy.go:218"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"started gRPC proxy"</span>,<span class="hljs-string">"address"</span>:<span class="hljs-string">"192.168.10.7:12379"</span>}
</code></pre>
<p data-nodeid="88828">可以看到，etcd gRPC proxy 启动后在<code data-backticks="1" data-nodeid="88935">192.168.10.7:12379</code>监听，并将客户端的请求转发到上述三个成员其中的一个。通过下述客户端读写命令，经过 proxy 发送请求：</p>
<pre class="lang-java" data-nodeid="88829"><code data-language="java">$ ETCDCTL_API=<span class="hljs-number">3</span> etcdctl --endpoints=<span class="hljs-number">192.168</span><span class="hljs-number">.10</span><span class="hljs-number">.7</span>:<span class="hljs-number">12379</span> put foo bar
OK
$ ETCDCTL_API=<span class="hljs-number">3</span> etcdctl --endpoints=<span class="hljs-number">192.168</span><span class="hljs-number">.10</span><span class="hljs-number">.7</span>:<span class="hljs-number">12379</span> get foo
foo
bar
</code></pre>
<p data-nodeid="88830">我们通过 grpc-proxy 提供的客户端地址进行访问，proxy 执行的结果符合预期，在使用方法上和普通的方式完全相同。</p>
<h3 data-nodeid="88831">客户端端点同步</h3>
<p data-nodeid="88832">gRPC 代理是 gRPC 命名的提供者，支持<strong data-nodeid="88944">在启动时通过写入相同的前缀端点名称</strong>进行注册。这样可以使客户端将其端点与具有一组相同前缀端点名的代理端点同步，进而实现高可用性。</p>
<p data-nodeid="88833">下面我们来启动两个 gRPC 代理，在启动时指定自定义的前缀<code data-backticks="1" data-nodeid="88946">___grpc_proxy_endpoint</code>来注册 gRPC 代理：</p>
<pre class="lang-java" data-nodeid="88834"><code data-language="java">$ etcd grpc-proxy start --endpoints=localhost:<span class="hljs-number">12379</span>   --listen-addr=<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">23790</span>   --advertise-client-url=<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">23790</span>   --resolver-prefix=<span class="hljs-string">"___grpc_proxy_endpoint"</span>   --resolver-ttl=<span class="hljs-number">60</span>
{<span class="hljs-string">"level"</span>:<span class="hljs-string">"info"</span>,<span class="hljs-string">"ts"</span>:<span class="hljs-string">"2020-12-13T01:46:04.885+0800"</span>,<span class="hljs-string">"caller"</span>:<span class="hljs-string">"etcdmain/grpc_proxy.go:320"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"listening for gRPC proxy client requests"</span>,<span class="hljs-string">"address"</span>:<span class="hljs-string">"127.0.0.1:23790"</span>}
{<span class="hljs-string">"level"</span>:<span class="hljs-string">"info"</span>,<span class="hljs-string">"ts"</span>:<span class="hljs-string">"2020-12-13T01:46:04.885+0800"</span>,<span class="hljs-string">"caller"</span>:<span class="hljs-string">"etcdmain/grpc_proxy.go:218"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"started gRPC proxy"</span>,<span class="hljs-string">"address"</span>:<span class="hljs-string">"127.0.0.1:23790"</span>}
<span class="hljs-number">2020</span>-<span class="hljs-number">12</span>-<span class="hljs-number">13</span> <span class="hljs-number">01</span>:<span class="hljs-number">46</span>:<span class="hljs-number">04.892061</span> I | grpcproxy: registered <span class="hljs-string">"127.0.0.1:23790"</span> with <span class="hljs-number">60</span>-second lease
$ etcd grpc-proxy start --endpoints=localhost:<span class="hljs-number">12379</span> \
&gt;   --listen-addr=<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">23791</span> \
&gt;   --advertise-client-url=<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">23791</span> \
&gt;   --resolver-prefix=<span class="hljs-string">"___grpc_proxy_endpoint"</span> \
&gt;   --resolver-ttl=<span class="hljs-number">60</span>
{<span class="hljs-string">"level"</span>:<span class="hljs-string">"info"</span>,<span class="hljs-string">"ts"</span>:<span class="hljs-string">"2020-12-13T01:46:43.616+0800"</span>,<span class="hljs-string">"caller"</span>:<span class="hljs-string">"etcdmain/grpc_proxy.go:320"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"listening for gRPC proxy client requests"</span>,<span class="hljs-string">"address"</span>:<span class="hljs-string">"127.0.0.1:23791"</span>}
{<span class="hljs-string">"level"</span>:<span class="hljs-string">"info"</span>,<span class="hljs-string">"ts"</span>:<span class="hljs-string">"2020-12-13T01:46:43.616+0800"</span>,<span class="hljs-string">"caller"</span>:<span class="hljs-string">"etcdmain/grpc_proxy.go:218"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"started gRPC proxy"</span>,<span class="hljs-string">"address"</span>:<span class="hljs-string">"127.0.0.1:23791"</span>}
<span class="hljs-number">2020</span>-<span class="hljs-number">12</span>-<span class="hljs-number">13</span> <span class="hljs-number">01</span>:<span class="hljs-number">46</span>:<span class="hljs-number">43.622249</span> I | grpcproxy: registered <span class="hljs-string">"127.0.0.1:23791"</span> with <span class="hljs-number">60</span>-second lease
</code></pre>
<p data-nodeid="88835">在上面的启动命令中，将需要加入的自定义端点<code data-backticks="1" data-nodeid="88949">--resolver-prefix</code>设置为<code data-backticks="1" data-nodeid="88951">___grpc_proxy_endpoint</code>。启动成功之后，我们来验证下，gRPC 代理在查询成员时是否列出其所有成员作为成员列表，执行如下的命令：</p>
<pre class="lang-java" data-nodeid="88836"><code data-language="java">ETCDCTL_API=<span class="hljs-number">3</span> etcdctl --endpoints=http:<span class="hljs-comment">//localhost:23790 member list --write-out table</span>
</code></pre>
<p data-nodeid="88837">通过下图，可以看到，通过相同的前缀端点名完成了自动发现所有成员列表的操作。</p>
<p data-nodeid="90475" class=""><img src="https://s0.lgstatic.com/i/image6/M00/02/39/CioPOWAdDuaAL9IoAAMgcPZE1jc101.png" alt="202125-92351.png" data-nodeid="90478"></p>



<p data-nodeid="88839">同样地，客户端也可以通过 Sync 方法自动发现代理的端点，代码实现如下：</p>
<pre class="lang-java" data-nodeid="88840"><code data-language="java">cli, err := clientv3.New(clientv3.Config{
    Endpoints: []string{<span class="hljs-string">"http://localhost:23790"</span>},
})
<span class="hljs-keyword">if</span> err != nil {
    log.Fatal(err)
}
defer cli.Close()
<span class="hljs-comment">// 获取注册过的 grpc-proxy 端点</span>
<span class="hljs-keyword">if</span> err := cli.Sync(context.Background()); err != nil {
    log.Fatal(err)
}
</code></pre>
<p data-nodeid="88841">相应地，如果配置的代理没有配置前缀，gRPC 代理启动命令如下：</p>
<pre class="lang-java" data-nodeid="88842"><code data-language="java">$ ./etcd grpc-proxy start --endpoints=localhost:12379 \
&gt;   --listen-addr=127.0.0.1:23792 \
&gt;   --advertise-client-url=127.0.0.1:23792
# 输出结果
{"level":"info","ts":"2020-12-13T01:49:25.099+0800","caller":"etcdmain/grpc_proxy.go:320","msg":"listening for gRPC proxy client requests","address":"127.0.0.1:23792"}
{"level":"info","ts":"2020-12-13T01:49:25.100+0800","caller":"etcdmain/grpc_proxy.go:218","msg":"started gRPC proxy","address":"127.0.0.1:23792"}
</code></pre>
<p data-nodeid="88843">我们来验证下 gRPC proxy 的成员列表 API 是不是只返回自己的<code data-backticks="1" data-nodeid="88960">advertise-client-url</code>：</p>
<pre class="lang-java" data-nodeid="88844"><code data-language="java">ETCDCTL_API=<span class="hljs-number">3</span> etcdctl --endpoints=http:<span class="hljs-comment">//localhost:23792 member list --write-out table</span>
</code></pre>
<p data-nodeid="88845">通过下图，可以看到，结果如我们预期：当我们<strong data-nodeid="88967">没有配置代理的前缀端点名时，获取其成员列表只会显示当前节点的信息，也不会包含其他的端点</strong>。</p>
<p data-nodeid="90943" class=""><img src="https://s0.lgstatic.com/i/image6/M00/02/3B/Cgp9HWAdDu2Afub8AAI6unk0A5A099.png" alt="202125-92353.png" data-nodeid="90946"></p>

<h3 data-nodeid="88847">可伸缩的 watch API</h3>
<p data-nodeid="88848">如果客户端监视同一键或某一范围内的键，gRPC 代理可以将这些客户端监视程序（c-watcher）合并为连接到 etcd 服务器的单个监视程序（s-watcher）。当 watch 事件发生时，代理将所有事件从 s-watcher 广播到其 c-watcher。</p>
<p data-nodeid="88849">假设 N 个客户端监视相同的 key，则 gRPC 代理可以将 etcd 服务器上的监视负载从 N 减少到 1。用户可以部署多个 gRPC 代理，进一步分配服务器负载。</p>
<p data-nodeid="88850">如下图所示，三个客户端监视键 A。gRPC 代理将三个监视程序合并，从而创建一个附加到 etcd 服务器的监视程序。</p>
<p data-nodeid="91411" class=""><img src="https://s0.lgstatic.com/i/image6/M00/02/39/CioPOWAdDvWAAhlsAADrPgju77A709.png" alt="202125-92355.png" data-nodeid="91414"></p>

<p data-nodeid="88852">为了有效地将多个客户端监视程序合并为一个监视程序，gRPC 代理在可能的情况下将新的 c-watcher 合并为现有的 s-watcher。由于网络延迟或缓冲的未传递事件，合并的 s-watcher 可能与 etcd 服务器不同步。</p>
<p data-nodeid="88853"><strong data-nodeid="88983">如果没有指定监视版本，gRPC 代理将不能保证 c-watcher 从最近的存储修订版本开始监视</strong>。例如，如果客户端从修订版本为 1000 的 etcd 服务器监视，则该监视者将从修订版本 1000 开始。如果客户端从 gRPC 代理监视，则可能从修订版本 990 开始监视。</p>
<p data-nodeid="88854"><strong data-nodeid="88988">类似的限制也适用于取消</strong>。取消 watch 后，etcd 服务器的修订版可能大于取消响应修订版。</p>
<p data-nodeid="88855">对于大多数情况，这两个限制一般不会引起问题，未来也可能会有其他选项强制观察者绕过 gRPC 代理以获得更准确的修订响应。</p>
<h3 data-nodeid="88856">可伸缩的 lease API</h3>
<p data-nodeid="88857">为了保持客户端申请租约的有效性，客户端至少建立一个 gRPC 连接到 etcd 服务器，以定期发送心跳信号。如果 etcd 工作负载涉及很多的客户端租约活动，这些流可能会导致 CPU 使用率过高。<strong data-nodeid="88996">为了减少核心集群上的流总数，gRPC 代理支持将 lease 流合并</strong>。</p>
<p data-nodeid="88858">假设有 N 个客户端正在更新租约，则单个 gRPC 代理将 etcd 服务器上的流负载从 N 减少到 1。在部署的过程中，可能还有其他 gRPC 代理，进一步在多个代理之间分配流。</p>
<p data-nodeid="88859">在下图示例中，三个客户端更新了三个独立的租约（L1、L2 和 L3）。gRPC 代理将三个客户端租约流（c-stream）合并为连接到 etcd 服务器的单个租约（s-stream），以保持活动流。代理将客户端租约的心跳从 c-stream 转发到 s-stream，然后将响应返回到相应的 c-stream。</p>
<p data-nodeid="91879" class=""><img src="https://s0.lgstatic.com/i/image6/M00/02/3C/Cgp9HWAdDwCAZYOCAAC-ZGY7vng236.png" alt="202125-92357.png" data-nodeid="91882"></p>

<p data-nodeid="88861">除此之外，gRPC 代理在满足一致性时会缓存请求的响应。该功能可以保护 etcd 服务器免遭恶意 for 循环中滥用客户端的攻击。</p>
<h3 data-nodeid="88862">命名空间的实现</h3>
<p data-nodeid="88863">上面我们讲到 gRPC proxy 的端点可以通过配置前缀，自动发现。而当应用程序期望对整个键空间有完全控制，etcd 集群与其他应用程序共享的情况下，为了使所有应用程序都不会相互干扰地运行，代理可以对<strong data-nodeid="89009">etcd 键空间进行分区</strong>，以便客户端大概率访问完整的键空间。</p>
<p data-nodeid="88864">当给代理提供标志<code data-backticks="1" data-nodeid="89011">--namespace</code>时，所有进入代理的客户端请求都将转换为<strong data-nodeid="89017">在键上具有用户定义的前缀</strong>。普通的请求对 etcd 集群的访问将会在我们指定的前缀（即指定的 --namespace 的值）下，而来自代理的响应将删除该前缀；而这个操作对于客户端来说是透明的，根本察觉不到前缀。</p>
<p data-nodeid="88865">下面我们给 gRPC proxy 命名，只需要启动时指定<code data-backticks="1" data-nodeid="89019">--namespace</code>标识：</p>
<pre class="lang-java" data-nodeid="88866"><code data-language="java">$ ./etcd grpc-proxy start --endpoints=localhost:<span class="hljs-number">12379</span> \
&gt;   --listen-addr=<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">23790</span> \
&gt;   --namespace=my-prefix/
{<span class="hljs-string">"level"</span>:<span class="hljs-string">"info"</span>,<span class="hljs-string">"ts"</span>:<span class="hljs-string">"2020-12-13T01:53:16.875+0800"</span>,<span class="hljs-string">"caller"</span>:<span class="hljs-string">"etcdmain/grpc_proxy.go:320"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"listening for gRPC proxy client requests"</span>,<span class="hljs-string">"address"</span>:<span class="hljs-string">"127.0.0.1:23790"</span>}
{<span class="hljs-string">"level"</span>:<span class="hljs-string">"info"</span>,<span class="hljs-string">"ts"</span>:<span class="hljs-string">"2020-12-13T01:53:16.876+0800"</span>,<span class="hljs-string">"caller"</span>:<span class="hljs-string">"etcdmain/grpc_proxy.go:218"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"started gRPC proxy"</span>,<span class="hljs-string">"address"</span>:<span class="hljs-string">"127.0.0.1:23790"</span>}
</code></pre>
<p data-nodeid="88867">此时对代理的访问会在 etcd 群集上自动地加上前缀，对于客户端来说没有感知。我们通过 etcdctl 客户端进行尝试：</p>
<pre class="lang-java" data-nodeid="88868"><code data-language="java">$ ETCDCTL_API=3 etcdctl --endpoints=localhost:23790 put my-key abc
# OK
$ ETCDCTL_API=3 etcdctl --endpoints=localhost:23790 get my-key
# my-key
# abc
$ ETCDCTL_API=3 etcdctl --endpoints=localhost:2379 get my-prefix/my-key
# my-prefix/my-key
# abc
</code></pre>
<p data-nodeid="88869">上述三条命令，首先通过代理写入键值对，然后读取。为了验证结果，第三条命令通过 etcd 集群直接读取，不过需要加上代理的前缀，两种方式得到的结果完全一致。因此，<strong data-nodeid="89027">使用 proxy 的命名空间即可实现 etcd 键空间分区</strong>，对于客户端来说非常便利。</p>
<h3 data-nodeid="88870">其他扩展功能</h3>
<p data-nodeid="88871">gRPC 代理的功能非常强大，除了上述提到的客户端端点同步、可伸缩 API、命名空间功能，还提供了指标与健康检查接口和 TLS 加密中止的扩展功能。</p>
<h4 data-nodeid="88872">指标与健康检查接口</h4>
<p data-nodeid="88873">gRPC 代理为<code data-backticks="1" data-nodeid="89032">--endpoints</code>定义的 etcd 成员公开了<code data-backticks="1" data-nodeid="89034">/health</code>和 Prometheus 的<code data-backticks="1" data-nodeid="89036">/metrics</code>接口。我们通过浏览器访问这两个接口：</p>
<p data-nodeid="88874"><img src="https://s0.lgstatic.com/i/image/M00/94/3D/Ciqc1GAXy8qAAw3iAAaZ1XYdHxw861.png" alt="Drawing 4.png" data-nodeid="89040"></p>
<div data-nodeid="88875"><p style="text-align:center">访问 metrics 接口的结果</p></div>
<p data-nodeid="88876"><img src="https://s0.lgstatic.com/i/image/M00/94/48/CgqCHmAXy8-AZ0q5AACKTo_Vhhg176.png" alt="Drawing 5.png" data-nodeid="89043"></p>
<div data-nodeid="88877"><p style="text-align:center">访问 health 接口的结果</p></div>
<p data-nodeid="88878">通过代理访问<code data-backticks="1" data-nodeid="89045">/metrics</code>端点的结果如上图所示 ，其实和普通的 etcd 集群实例没有什么区别，同样也会结合一些中间件进行统计和页面展示，如 Prometheus 和 Grafana 的组合。</p>
<p data-nodeid="88879">除了使用默认的端点访问这两个接口，另一种方法是定义一个附加 URL，该 URL 将通过&nbsp;--metrics-addr&nbsp;标志来响应<code data-backticks="1" data-nodeid="89048">/metrics</code>和<code data-backticks="1" data-nodeid="89050">/health</code>端点。命令如下所示 ：</p>
<pre class="lang-java" data-nodeid="88880"><code data-language="java">$ ./etcd grpc-proxy start \
  --endpoints http:<span class="hljs-comment">//localhost:12379 \</span>
  --metrics-addr http:<span class="hljs-comment">//0.0.0.0:6633 \</span>
  --listen-addr <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">23790</span> \
</code></pre>
<p data-nodeid="88881">在执行如上启动命令时，会有如下的命令行输出，提示我们指定的 metrics 监听地址为 http://0.0.0.0:6633。</p>
<pre class="lang-powershell" data-nodeid="88882"><code data-language="powershell">{<span class="hljs-string">"level"</span>:<span class="hljs-string">"info"</span>,<span class="hljs-string">"ts"</span>:<span class="hljs-string">"2021-01-30T18:03:45.231+0800"</span>,<span class="hljs-string">"caller"</span>:<span class="hljs-string">"etcdmain/grpc_proxy.go:456"</span>,<span class="hljs-string">"msg"</span>:<span class="hljs-string">"gRPC proxy listening for metrics"</span>,<span class="hljs-string">"address"</span>:<span class="hljs-string">"http://0.0.0.0:6633"</span>}
</code></pre>
<h4 data-nodeid="88883">TLS 加密的代理</h4>
<p data-nodeid="88884">通过使用 gRPC 代理 etcd 集群的 TLS，可以给没有使用 HTTPS 加密方式的本地客户端提供服务，实现 etcd 集群的 TLS 加密中止，即未加密的客户端与 gRPC 代理通过 HTTP 方式通信，gRPC 代理与 etcd 集群通过 TLS 加密通信。下面我们进行实践：</p>
<pre class="lang-java" data-nodeid="88885"><code data-language="java">$ etcd --listen-client-urls https:<span class="hljs-comment">//localhost:12379 --advertise-client-urls https://localhost:2379 --cert-file=peer.crt --key-file=peer.key --trusted-ca-file=ca.crt --client-cert-auth</span>
</code></pre>
<p data-nodeid="88886">上述命令使用 HTTPS 启动了单个成员的 etcd 集群，然后确认 etcd 集群以 HTTPS 的方式提供服务：</p>
<pre class="lang-java" data-nodeid="88887"><code data-language="java"># fails
$ ETCDCTL_API=3 etcdctl --endpoints=http://localhost:2379 endpoint status
# works
$ ETCDCTL_API=3 etcdctl --endpoints=https://localhost:2379 --cert=client.crt --key=client.key --cacert=ca.crt endpoint status
</code></pre>
<p data-nodeid="88888">显然第一种方式不能访问。</p>
<p data-nodeid="88889">接下来通过使用客户端证书连接到 etcd 端点<code data-backticks="1" data-nodeid="89058">https://localhost:2379</code>，并在&nbsp;localhost:12379&nbsp;上启动 gRPC 代理，命令如下：</p>
<pre class="lang-java" data-nodeid="88890"><code data-language="java">$ etcd grpc-proxy start --endpoints=https:<span class="hljs-comment">//localhost:2379 --listen-addr localhost:12379 --cert client.crt --key client.key --cacert=ca.crt --insecure-skip-tls-verify</span>
</code></pre>
<p data-nodeid="88891">启动后，我们通过 gRPC 代理写入一个键值对测试：</p>
<pre class="lang-java" data-nodeid="88892"><code data-language="java">$ ETCDCTL_API=3 etcdctl --endpoints=http://localhost:12379 put abc def
# OK
</code></pre>
<p data-nodeid="88893">可以看到，使用 HTTP 的方式设置成功。</p>
<p data-nodeid="88894">回顾上述操作，我们通过 etcd 的 gRPC 代理实现了代理与实际的 etcd 集群之间的 TLS 加密，而本地的客户端通过 HTTP 的方式与 gRPC 代理通信。因此这是一个简便的调试和开发手段，你在生产环境需要谨慎使用，以防安全风险。</p>
<h3 data-nodeid="88895">小结</h3>
<p data-nodeid="88896">这一讲我们主要介绍了 etcd 中的 gRPC proxy。本讲主要内容如下：</p>
<p data-nodeid="92347" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/02/39/CioPOWAdDxKAFk1BAAIimkNUiSk285.png" alt="202125-92359.png" data-nodeid="92350"></p>

<p data-nodeid="88898">gRPC 代理用于支持多个 etcd 服务器端点，当代理启动时，它会随机选择一个 etcd 服务器端点来使用，该端点处理所有请求，直到代理检测到端点故障为止。如果 gRPC 代理检测到端点故障，它将切换到其他可用的端点，对客户端继续提供服务，并且隐藏了存在问题的 etcd 服务端点。</p>
<p data-nodeid="88899">关于 gRPC 代理，你有什么经验和踩坑的经历，欢迎在留言区和我分享你的经验。</p>
<p data-nodeid="88900" class="">集群的部署并不是一劳永逸的事情，在我们日常的工作中经常会遇到集群的调整。下一讲，我们将会介绍如何动态配置 etcd 集群。我们下一讲再见。</p>

---

### 精选评论

##### **琪：
> 为什么要固定一个etcd服务器使用，既然是集群每次都随机选择一个etcd服务器使用不是更好吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; gRPC proxy 提供的特性看下哈。

##### *冬：
> 老师你好。本质上grpc-Gateway 和 grpc-proxy 怎么感觉差不多呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 仔细看下区别哈，其实是不一样的使用场景。

