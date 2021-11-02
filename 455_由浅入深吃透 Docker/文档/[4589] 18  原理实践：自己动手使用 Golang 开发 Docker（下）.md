<p data-nodeid="210311">上一课时我们安装了 Golang，学习了一些容器必备的基础知识，并且自己动手编译了一个 gocker，实现了 Namespace 的隔离。今天我将带你深入剖析 gocker 的源码和实现原理，并且带你实现 cgroups 的资源限制。</p>



<h3 data-nodeid="209582">gocker 源码剖析</h3>
<p data-nodeid="209583">打开 gocker 的源码，我们可以看到 gocker 的实现主要有两个 go 文件：一个是 main.go，一个是 run.go。这两个文件起了什么作用呢？</p>
<p data-nodeid="209584">我们首先来看下 main.go 文件：</p>
<pre class="lang-go" data-nodeid="209585"><code data-language="go">$ cat main.<span class="hljs-keyword">go</span>
<span class="hljs-keyword">package</span> main

<span class="hljs-keyword">import</span> (
    <span class="hljs-string">"log"</span>
    <span class="hljs-string">"os"</span>

    <span class="hljs-string">"github.com/urfave/cli/v2"</span>
    <span class="hljs-string">"github.com/wilhelmguo/gocker/runc"</span>
)

<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
    app := cli.NewApp()
    app.Name = <span class="hljs-string">"gocker"</span>
    app.Usage = <span class="hljs-string">"gocker 是 golang 编写的精简版 Docker，目的是学习 Docker 的运行原理。"</span>

    app.Commands = []*cli.Command{
        runc.InitCommand,
        runc.RunCommand,
    }

    <span class="hljs-keyword">if</span> err := app.Run(os.Args); err != <span class="hljs-literal">nil</span> {
        log.Fatal(err)
    }
}
</code></pre>
<p data-nodeid="209586">main.go 文件中引用了一个第三方工具库 github.com/urfave/cli，该工具库提供了一个编写命令行的工具，可以帮助我们快速构建命令行应用程序，Docker 默认的容器运行时 runC 也引用了该工具库。<br>
main 函数是 gocker 执行的入口文件，main 定义了 gocker 的名称和简单介绍，同时调用了 InitCommand 和 RunCommand 实现了<code data-backticks="1" data-nodeid="209720">gocker init</code>和<code data-backticks="1" data-nodeid="209722">gocker run</code>这两个命令的初始化。</p>
<p data-nodeid="209587">下面我们查看一下 run.go 的文件内容，run.go 文件中定义了 InitCommand 和 RunCommand 的详细实现以及容器启动的过程，文件内容如下。</p>
<pre class="lang-go" data-nodeid="209588"><code data-language="go">$ cat runc/run.<span class="hljs-keyword">go</span>
<span class="hljs-keyword">package</span> runc

<span class="hljs-keyword">import</span> (
    <span class="hljs-string">"errors"</span>
    <span class="hljs-string">"fmt"</span>
    <span class="hljs-string">"io/ioutil"</span>
    <span class="hljs-string">"log"</span>
    <span class="hljs-string">"os"</span>
    <span class="hljs-string">"os/exec"</span>
    <span class="hljs-string">"path/filepath"</span>
    <span class="hljs-string">"strings"</span>
    <span class="hljs-string">"syscall"</span>

    <span class="hljs-string">"github.com/urfave/cli/v2"</span>
)

<span class="hljs-keyword">var</span> RunCommand = &amp;cli.Command{
    Name: <span class="hljs-string">"run"</span>,
    Usage: <span class="hljs-string">`启动一个隔离的容器
            gocker run -it [command]`</span>,
    Flags: []cli.Flag{
        &amp;cli.BoolFlag{
            Name:  <span class="hljs-string">"it"</span>,
            Usage: <span class="hljs-string">"是否启用命令行交互模式"</span>,
        },
        &amp;cli.StringFlag{
            Name:  <span class="hljs-string">"rootfs"</span>,
            Usage: <span class="hljs-string">"容器根目录"</span>,
        },
    },
    Action: <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(context *cli.Context)</span> <span class="hljs-title">error</span></span> {
        <span class="hljs-keyword">if</span> context.Args().Len() &lt; <span class="hljs-number">1</span> {
            <span class="hljs-keyword">return</span> errors.New(<span class="hljs-string">"参数不全，请检查！"</span>)
        }
        read, write, err := os.Pipe()
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            <span class="hljs-keyword">return</span> err
        }
        tty := context.Bool(<span class="hljs-string">"it"</span>)
        rootfs := context.String(<span class="hljs-string">"rootfs"</span>)
        
        cmd := exec.Command(<span class="hljs-string">"/proc/self/exe"</span>, <span class="hljs-string">"init"</span>)
        cmd.SysProcAttr = &amp;syscall.SysProcAttr{
            Cloneflags: syscall.CLONE_NEWNS |
                syscall.CLONE_NEWUTS |
                syscall.CLONE_NEWIPC |
                syscall.CLONE_NEWPID |
                syscall.CLONE_NEWNET,
        }
        <span class="hljs-keyword">if</span> tty {
            cmd.Stdin = os.Stdin
            cmd.Stdout = os.Stdout
            cmd.Stderr = os.Stderr
        }
        cmd.ExtraFiles = []*os.File{read}
        cmd.Dir = rootfs
        <span class="hljs-keyword">if</span> err := cmd.Start(); err != <span class="hljs-literal">nil</span> {
            log.Println(<span class="hljs-string">"command start error"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        write.WriteString(strings.Join(context.Args().Slice(), <span class="hljs-string">" "</span>))
        write.Close()
        cmd.Wait()
        <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
    },
}

<span class="hljs-keyword">var</span> InitCommand = &amp;cli.Command{
    Name:  <span class="hljs-string">"init"</span>,
    Usage: <span class="hljs-string">"初始化容器进程，请勿直接调用！"</span>,
    Action: <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(context *cli.Context)</span> <span class="hljs-title">error</span></span> {
        pwd, err := os.Getwd()
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            log.Printf(<span class="hljs-string">"Get current path error %v"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        log.Println(<span class="hljs-string">"Current path is "</span>, pwd)
        cmdArray := readCommandArray()
        <span class="hljs-keyword">if</span> cmdArray == <span class="hljs-literal">nil</span> || <span class="hljs-built_in">len</span>(cmdArray) == <span class="hljs-number">0</span> {
            <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"Command is empty"</span>)
        }
        log.Println(<span class="hljs-string">"CmdArray is "</span>, cmdArray)
        err = pivotRoot(pwd)
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            log.Printf(<span class="hljs-string">"pivotRoot error %v"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        <span class="hljs-comment">//mount proc</span>
        defaultMountFlags := syscall.MS_NOEXEC | syscall.MS_NOSUID | syscall.MS_NODEV
        syscall.Mount(<span class="hljs-string">"proc"</span>, <span class="hljs-string">"/proc"</span>, <span class="hljs-string">"proc"</span>, <span class="hljs-keyword">uintptr</span>(defaultMountFlags), <span class="hljs-string">""</span>)

        <span class="hljs-comment">// 配置hostname</span>
        <span class="hljs-keyword">if</span> err := syscall.Sethostname([]<span class="hljs-keyword">byte</span>(<span class="hljs-string">"lagoudocker"</span>)); err != <span class="hljs-literal">nil</span> {
            fmt.Printf(<span class="hljs-string">"Error setting hostname - %s\n"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        path, err := exec.LookPath(cmdArray[<span class="hljs-number">0</span>])
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            log.Printf(<span class="hljs-string">"Exec loop path error %v"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        <span class="hljs-comment">// export PATH=$PATH:/bin</span>
        <span class="hljs-keyword">if</span> err := syscall.Exec(path, cmdArray[<span class="hljs-number">0</span>:], os.Environ()); err != <span class="hljs-literal">nil</span> {
            log.Println(err.Error())
        }
        <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
    },
}

<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">pivotRoot</span><span class="hljs-params">(root <span class="hljs-keyword">string</span>)</span> <span class="hljs-title">error</span></span> {
    <span class="hljs-comment">// 确保新 root 和老 root 不在同一目录</span>
    <span class="hljs-comment">// MS_BIND：执行bind挂载，使文件或者子目录树在文件系统内的另一个点上可视。</span>
    <span class="hljs-comment">// MS_REC： 创建递归绑定挂载，递归更改传播类型</span>
    <span class="hljs-keyword">if</span> err := syscall.Mount(root, root, <span class="hljs-string">"bind"</span>, syscall.MS_BIND|syscall.MS_REC, <span class="hljs-string">""</span>); err != <span class="hljs-literal">nil</span> {
        <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"Mount rootfs to itself error: %v"</span>, err)
    }
    <span class="hljs-comment">// 创建 .pivot_root 文件夹，用于存储 old_root</span>
    pivotDir := filepath.Join(root, <span class="hljs-string">".pivot_root"</span>)
    <span class="hljs-keyword">if</span> err := os.Mkdir(pivotDir, <span class="hljs-number">0777</span>); err != <span class="hljs-literal">nil</span> {
        <span class="hljs-keyword">return</span> err
    }
    <span class="hljs-comment">// 调用 Golang 封装的 PivotRoot</span>
    <span class="hljs-keyword">if</span> err := syscall.PivotRoot(root, pivotDir); err != <span class="hljs-literal">nil</span> {
        <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"pivot_root %v"</span>, err)
    }
    <span class="hljs-comment">// 修改工作目录</span>
    <span class="hljs-keyword">if</span> err := syscall.Chdir(<span class="hljs-string">"/"</span>); err != <span class="hljs-literal">nil</span> {
        <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"chdir / %v"</span>, err)
    }
    pivotDir = filepath.Join(<span class="hljs-string">"/"</span>, <span class="hljs-string">".pivot_root"</span>)
    <span class="hljs-comment">// 卸载 .pivot_root</span>
    <span class="hljs-keyword">if</span> err := syscall.Unmount(pivotDir, syscall.MNT_DETACH); err != <span class="hljs-literal">nil</span> {
        <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"unmount pivot_root dir %v"</span>, err)
    }
    <span class="hljs-comment">// 删除临时文件夹 .pivot_root</span>
    <span class="hljs-keyword">return</span> os.Remove(pivotDir)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">readCommandArray</span><span class="hljs-params">()</span> []<span class="hljs-title">string</span></span> {
    pipe := os.NewFile(<span class="hljs-keyword">uintptr</span>(<span class="hljs-number">3</span>), <span class="hljs-string">"pipe"</span>)
    msg, err := ioutil.ReadAll(pipe)
    <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
        log.Printf(<span class="hljs-string">"init read pipe error %v"</span>, err)
        <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
    }
    msgStr := <span class="hljs-keyword">string</span>(msg)
    <span class="hljs-keyword">return</span> strings.Split(msgStr, <span class="hljs-string">" "</span>)
}

</code></pre>
<p data-nodeid="210793">看到这么多代码你是不是有点懵？别担心，我帮你一一解读。</p>
<p data-nodeid="210794">上面文件中有两个比较重要的变量 InitCommand 和 RunCommand，它们的作用如下：</p>

<ul data-nodeid="209590">
<li data-nodeid="209591">
<p data-nodeid="209592">RunCommand 是当我们执行 gocker run 命令时调用的函数，是实现 gocker run 的入口；</p>
</li>
<li data-nodeid="209593">
<p data-nodeid="209594">InitCommand 是当我们执行 gocker run 时自动调用 gocker init 来初始化容器的一些环境。</p>
</li>
</ul>
<h4 data-nodeid="209595">RunCommand （容器启动的入口）</h4>
<p data-nodeid="209596">我们先从 RunCommand 来分析：</p>
<pre class="lang-go" data-nodeid="209597"><code data-language="go"><span class="hljs-keyword">var</span> RunCommand = &amp;cli.Command{
    <span class="hljs-comment">// 定义一个启动命令，这里定义的是 run 命令，当执行 gocker run 时会调用该函数</span>
    Name: <span class="hljs-string">"run"</span>,
    <span class="hljs-comment">// 使用说明</span>
    Usage: <span class="hljs-string">`启动一个隔离的容器
            gocker run -it [command]`</span>,
    <span class="hljs-comment">// 执行 gocker run 命令可以传递的参数</span>
    Flags: []cli.Flag{
        &amp;cli.BoolFlag{
            Name:  <span class="hljs-string">"it"</span>,
            Usage: <span class="hljs-string">"是否启用命令行交互模式"</span>,
        },
        &amp;cli.StringFlag{
            Name:  <span class="hljs-string">"rootfs"</span>,
            Usage: <span class="hljs-string">"容器根目录"</span>,
        },
    },
    <span class="hljs-comment">// gocker run 命令的执行函数</span>
    Action: <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(context *cli.Context)</span> <span class="hljs-title">error</span></span> {
        <span class="hljs-comment">// 校验参数 </span>
        <span class="hljs-keyword">if</span> context.Args().Len() &lt; <span class="hljs-number">1</span> {
            <span class="hljs-keyword">return</span> errors.New(<span class="hljs-string">"参数不全，请检查！"</span>)
        }
        read, write, err := os.Pipe()
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            <span class="hljs-keyword">return</span> err
        }
        <span class="hljs-comment">// 获取传入的参数的值</span>
        tty := context.Bool(<span class="hljs-string">"it"</span>)
        rootfs := context.String(<span class="hljs-string">"rootfs"</span>)
        <span class="hljs-comment">// 这里执行 /proc/self/exe init 相当于执行 gocker init</span>
        cmd := exec.Command(<span class="hljs-string">"/proc/self/exe"</span>, <span class="hljs-string">"init"</span>)
        <span class="hljs-comment">// 定义新创建哪些命名空间</span>
        cmd.SysProcAttr = &amp;syscall.SysProcAttr{
            Cloneflags: syscall.CLONE_NEWNS |
                syscall.CLONE_NEWUTS |
                syscall.CLONE_NEWIPC |
                syscall.CLONE_NEWPID |
                syscall.CLONE_NEWNET,
        }
        <span class="hljs-comment">// 把容器的标准输出重定向到主机的标准输出</span>
        <span class="hljs-keyword">if</span> tty {
            cmd.Stdin = os.Stdin
            cmd.Stdout = os.Stdout
            cmd.Stderr = os.Stderr
        }
        cmd.ExtraFiles = []*os.File{read}
        cmd.Dir = rootfs
        <span class="hljs-comment">// 启动容器</span>
        <span class="hljs-keyword">if</span> err := cmd.Start(); err != <span class="hljs-literal">nil</span> {
            log.Println(<span class="hljs-string">"command start error"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        write.WriteString(strings.Join(context.Args().Slice(), <span class="hljs-string">" "</span>))
        write.Close()
        <span class="hljs-comment">// 等待容器退出</span>
        cmd.Wait()
        <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
        }
</code></pre>
<p data-nodeid="209598">RunCommand 变量实际上是一个 Command 结构体，这个结构体包含了四个变量。</p>
<ol data-nodeid="209599">
<li data-nodeid="209600">
<p data-nodeid="209601">Name：定义一个启动命令，这里定义的是 run 命令，当执行 gocker run 时会调用该函数。</p>
</li>
<li data-nodeid="209602">
<p data-nodeid="209603">Usage：<code data-backticks="1" data-nodeid="209735">gocker run</code>命令的使用说明。</p>
</li>
<li data-nodeid="209604">
<p data-nodeid="209605">Flags：执行<code data-backticks="1" data-nodeid="209738">gocker run</code>命令可以传递的参数。</p>
</li>
<li data-nodeid="209606">
<p data-nodeid="209607">Action： 该变量是真正的 gocker run 命令的入口， 主要做了以下事情：</p>
<ul data-nodeid="209608">
<li data-nodeid="209609">
<p data-nodeid="209610">校验 gocker run 传递的参数；</p>
</li>
<li data-nodeid="209611">
<p data-nodeid="209612">构造一个 Pipe，把 gocker 的启动参数写入，方便在 init 进程中获取；</p>
</li>
<li data-nodeid="209613">
<p data-nodeid="209614">定义 /proc/self/exe init 调用，相当于调用 gocker init ；</p>
</li>
<li data-nodeid="209615">
<p data-nodeid="209616">创建五种命名空间用于资源隔离，分别为 Mount Namespace、UTS Namespace、IPC Namespace、PID Namespace 和 Net Namespace；</p>
</li>
<li data-nodeid="209617">
<p data-nodeid="209618">调用 cmd.Start 函数，开始执行容器启动步骤，首先创建出来一个 namespace （上一步定义的五种namespace）隔离的进程，然后调用 /proc/self/exe，也就是调用 gocker init，执行 InitCommand 中定义的容器初始化步骤。</p>
</li>
</ul>
</li>
</ol>
<p data-nodeid="209619">那么 InitCommand 究竟做了什么呢？</p>
<h4 data-nodeid="209620">InitCommand（准备容器环境）</h4>
<p data-nodeid="209621">下面我们看下 InitCommand 中的内容：</p>
<pre class="lang-go" data-nodeid="209622"><code data-language="go"><span class="hljs-keyword">var</span> InitCommand = &amp;cli.Command{
    Name:  <span class="hljs-string">"init"</span>,
    Usage: <span class="hljs-string">"初始化容器进程，请勿直接调用！"</span>,
    Action: <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(context *cli.Context)</span> <span class="hljs-title">error</span></span> {
        <span class="hljs-comment">// 获取当前执行目录</span>
        pwd, err := os.Getwd()
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            log.Printf(<span class="hljs-string">"Get current path error %v"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        log.Println(<span class="hljs-string">"Current path is "</span>, pwd)
        <span class="hljs-comment">// 获取用户传递的启动参数</span>
        cmdArray := readCommandArray()
        <span class="hljs-keyword">if</span> cmdArray == <span class="hljs-literal">nil</span> || <span class="hljs-built_in">len</span>(cmdArray) == <span class="hljs-number">0</span> {
            <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"Command is empty"</span>)
        }
        log.Println(<span class="hljs-string">"CmdArray is "</span>, cmdArray)
        <span class="hljs-comment">// pivotRoot 的作用类似于 chroot，可以把我们准备的镜像目录设置为容器的根目录。</span>
        err = pivotRoot(pwd)
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            log.Printf(<span class="hljs-string">"pivotRoot error %v"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        <span class="hljs-comment">// 挂载容器自己的 proc 目录，实现 ps 只能看到容器自己的进程</span>
        defaultMountFlags := syscall.MS_NOEXEC | syscall.MS_NOSUID | syscall.MS_NODEV
        syscall.Mount(<span class="hljs-string">"proc"</span>, <span class="hljs-string">"/proc"</span>, <span class="hljs-string">"proc"</span>, <span class="hljs-keyword">uintptr</span>(defaultMountFlags), <span class="hljs-string">""</span>)
        <span class="hljs-comment">// 配置主机名为 lagoudocker</span>
        <span class="hljs-keyword">if</span> err := syscall.Sethostname([]<span class="hljs-keyword">byte</span>(<span class="hljs-string">"lagoudocker"</span>)); err != <span class="hljs-literal">nil</span> {
            fmt.Printf(<span class="hljs-string">"Error setting hostname - %s\n"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        path, err := exec.LookPath(cmdArray[<span class="hljs-number">0</span>])
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            log.Printf(<span class="hljs-string">"Exec loop path error %v"</span>, err)
            <span class="hljs-keyword">return</span> err
        }
        <span class="hljs-comment">// syscall.Exec 相当于 shell 中的 exec 实现，这里用 用户传递的主命令来替换 init 进程，从而实现容器的 1 号进程为用户传递的主进程</span>
        <span class="hljs-keyword">if</span> err := syscall.Exec(path, cmdArray[<span class="hljs-number">0</span>:], os.Environ()); err != <span class="hljs-literal">nil</span> {
            log.Println(err.Error())
        }
        <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
    },
}
      
</code></pre>
<p data-nodeid="211277">通过代码你能看出 InitCommand 都做了哪些容器启动前的准备工作吗？</p>
<p data-nodeid="211278">InitCommand 主要做了以下几件事情：</p>

<ol data-nodeid="209624">
<li data-nodeid="209625">
<p data-nodeid="209626">获取当前运行目录；</p>
</li>
<li data-nodeid="209627">
<p data-nodeid="209628">从 RunCommand 中获取用户传递的容器启动参数；</p>
</li>
<li data-nodeid="209629">
<p data-nodeid="209630">修改当前进程运行的根目录为用户传递的 rootfs 目录；</p>
</li>
<li data-nodeid="209631">
<p data-nodeid="209632">挂载容器自己的 proc 目录，使得容器中执行 ps 命令只能看到自己命名空间下的进程；</p>
</li>
<li data-nodeid="209633">
<p data-nodeid="209634">设置容器的主机名称为 lagoudocker；</p>
</li>
<li data-nodeid="209635">
<p data-nodeid="209636">执行 syscall.Exec 实现使用用户传递的启动命令替换当前 init 进程。</p>
</li>
</ol>
<p data-nodeid="209637">这里有两个比较关键的技术点 pivotRoot 和 syscall.Exec。</p>
<ul data-nodeid="209638">
<li data-nodeid="209639">
<p data-nodeid="209640">pivotRoot：pivotRoot 是一个系统调用，主要功能是改变当前进程的根目录，它可以把当前进程的根目录移动到我们传递的 rootfs 目录下，从而使得我们不仅能够看到指定目录，还可以看到它的子目录信息。</p>
</li>
<li data-nodeid="209641">
<p data-nodeid="209642" class="">syscall.Exec：syscall.Exec 是一个系统调用，这个系统调用可以实现执行指定的命令，但是并不创建新的进程，而是在当前的进程空间执行，替换掉正在执行的进程，复用同一个进程号。通过这种机制，才实现了我们在容器中看到的 1 号进程是我们传递的命令，而不是 init 进程。</p>
</li>
</ul>
<p data-nodeid="209643">最后，总结下容器的完整创建流程:</p>
<p data-nodeid="211761" class="">1.使用以下命令创建容器</p>
<pre class="lang-plain" data-nodeid="211762"><code data-language="plain">gocker run -it -rootfs=/tmp/busybox /bin/sh
</code></pre>
<p data-nodeid="214259">2.RunCommand 解析请求的参数（-it -rootfs=/tmp/busybox）和主进程启动命令（/bin/sh）；</p>
<p data-nodeid="214733">3.创建 namespace 隔离的容器进程；</p>
<p data-nodeid="215205">4.启动容器进程；</p>
<p data-nodeid="215206">5.容器内的进程执行 /proc/self/exe 调用自己实现容器的初始化，修改当前进程运行的根目录，挂载 proc 文件系统，修改主机名，最后使用 sh 进程替换当前容器的进程，使得容器的主进程为 sh 进程。</p>







<p data-nodeid="211772">目前我们的容器虽然实现了使用 Namespace 隔离各种资源，但是容器内的进程仍然可以任意地使用主机的 CPU 、内存等资源。而这可能导致主机的资源竞争，下面我们使用cgroups来实现对 CPU 和内存的限制。</p>
<h3 data-nodeid="211773">为 gocker 添加 cgroups 限制</h3>
<p data-nodeid="211774"><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=455#/detail/pc?id=4581" data-nodeid="211835">在第 10 讲中</a>，我们手动操作 cgroups 实现了对容器资源的限制，下面我把这部分手动操作转化为代码。</p>
<h4 data-nodeid="211775">编写资源限制源码</h4>
<p data-nodeid="211776">首先我们定义 cgroups 的挂载目录和我们要创建的目录，定义如下：</p>
<pre class="lang-go" data-nodeid="211777"><code data-language="go"><span class="hljs-keyword">const</span> gockerCgroupPath = <span class="hljs-string">"gocker"</span>
<span class="hljs-keyword">const</span> cgroupsRoot = <span class="hljs-string">"/sys/fs/cgroup"</span>
</code></pre>
<p data-nodeid="211778">然后定义Cgroups结构体，分别定义 CPU 和 Memory 字段，用于存储用户端传递的 CPU 和 Memory 限制值：</p>
<pre class="lang-go" data-nodeid="211779"><code data-language="go"><span class="hljs-keyword">type</span> Cgroups <span class="hljs-keyword">struct</span> {
    <span class="hljs-comment">// 单位 核</span>
    CPU <span class="hljs-keyword">int</span>
    <span class="hljs-comment">// 单位 兆</span>
    Memory <span class="hljs-keyword">int</span>
}
</code></pre>
<p data-nodeid="211780">接着定义 Cgroups 对象的一些操作方法，这样方便我们对当前容器的 cgroups 进程操作。方法定义如下。</p>
<ul data-nodeid="211781">
<li data-nodeid="211782">
<p data-nodeid="211783">Apply：把容器的 pid 写入到对应子系统下的 tasks 文件中，使得 cgroups 限制对容器进程生效。</p>
</li>
<li data-nodeid="211784">
<p data-nodeid="211785">Destroy：容器退出时删除对应的 cgroups 文件。</p>
</li>
<li data-nodeid="211786">
<p data-nodeid="211787">SetCPULimit：将 CPU 限制值写入到 cpu.cfs_quota_us 文件中。</p>
</li>
<li data-nodeid="211788">
<p data-nodeid="211789">SetMemoryLimit：将内存限制值写入 memory.limit_in_bytes 文件中。</p>
</li>
</ul>
<pre class="lang-go" data-nodeid="211790"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(c *Cgroups)</span> <span class="hljs-title">Apply</span><span class="hljs-params">(pid <span class="hljs-keyword">int</span>)</span> <span class="hljs-title">error</span></span> {
    <span class="hljs-keyword">if</span> c.CPU != <span class="hljs-number">0</span> {
        cpuCgroupPath, err := getCgroupPath(<span class="hljs-string">"cpu"</span>, <span class="hljs-literal">true</span>)
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            <span class="hljs-keyword">return</span> err
        }
        err = ioutil.WriteFile(path.Join(cpuCgroupPath, <span class="hljs-string">"tasks"</span>), []<span class="hljs-keyword">byte</span>(strconv.Itoa(pid)), <span class="hljs-number">0644</span>)
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"set cgroup cpu fail %v"</span>, err)
        }
    }
    <span class="hljs-keyword">if</span> c.Memory != <span class="hljs-number">0</span> {
        memoryCgroupPath, err := getCgroupPath(<span class="hljs-string">"memory"</span>, <span class="hljs-literal">true</span>)
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            <span class="hljs-keyword">return</span> err
        }
        err = ioutil.WriteFile(path.Join(memoryCgroupPath, <span class="hljs-string">"tasks"</span>), []<span class="hljs-keyword">byte</span>(strconv.Itoa(pid)), <span class="hljs-number">0644</span>)
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"set cgroup memory fail %v"</span>, err)
        }
    }
    <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
}

<span class="hljs-comment">// 释放cgroup</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(c *Cgroups)</span> <span class="hljs-title">Destroy</span><span class="hljs-params">()</span> <span class="hljs-title">error</span></span> {
    <span class="hljs-keyword">if</span> c.CPU != <span class="hljs-number">0</span> {
        cpuCgroupPath, err := getCgroupPath(<span class="hljs-string">"cpu"</span>, <span class="hljs-literal">false</span>)
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            <span class="hljs-keyword">return</span> err
        }
        <span class="hljs-keyword">return</span> os.RemoveAll(cpuCgroupPath)
    }
    <span class="hljs-keyword">if</span> c.Memory != <span class="hljs-number">0</span> {
        memoryCgroupPath, err := getCgroupPath(<span class="hljs-string">"memory"</span>, <span class="hljs-literal">false</span>)
        <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            <span class="hljs-keyword">return</span> err
        }
        <span class="hljs-keyword">return</span> os.RemoveAll(memoryCgroupPath)
    }
    <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
}

<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(c *Cgroups)</span> <span class="hljs-title">SetCPULimit</span><span class="hljs-params">(cpu <span class="hljs-keyword">int</span>)</span> <span class="hljs-title">error</span></span> {
    cpuCgroupPath, err := getCgroupPath(<span class="hljs-string">"cpu"</span>, <span class="hljs-literal">true</span>)
    <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
        <span class="hljs-keyword">return</span> err
    }
    <span class="hljs-keyword">if</span> err := ioutil.WriteFile(path.Join(cpuCgroupPath, <span class="hljs-string">"cpu.cfs_quota_us"</span>), []<span class="hljs-keyword">byte</span>(strconv.Itoa(cpu*<span class="hljs-number">100000</span>)), <span class="hljs-number">0644</span>); err != <span class="hljs-literal">nil</span> {
        <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"set cpu limit fail %v"</span>, err)
    }
    <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
}

<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(c *Cgroups)</span> <span class="hljs-title">SetMemoryLimit</span><span class="hljs-params">(memory <span class="hljs-keyword">int</span>)</span> <span class="hljs-title">error</span></span> {
    memoryCgroupPath, err := getCgroupPath(<span class="hljs-string">"memory"</span>, <span class="hljs-literal">true</span>)
    <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
        <span class="hljs-keyword">return</span> err
    }
    <span class="hljs-keyword">if</span> err := ioutil.WriteFile(path.Join(memoryCgroupPath, <span class="hljs-string">"memory.limit_in_bytes"</span>), []<span class="hljs-keyword">byte</span>(strconv.Itoa(memory*<span class="hljs-number">1024</span>*<span class="hljs-number">1024</span>)), <span class="hljs-number">0644</span>); err != <span class="hljs-literal">nil</span> {
        <span class="hljs-keyword">return</span> fmt.Errorf(<span class="hljs-string">"set memory limit fail %v"</span>, err)
    }
    <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
}
</code></pre>
<p data-nodeid="211791">最后在 run 命令的 Action 函数中，添加 cgroups 初始化逻辑，将 CPU 和内存的限制值写入到 cgroups 文件中，并且将当前进程的 pid 也写入到 cgroups 的 tasks 文件中，使得 CPU 和内存的限制对于当前容器进程生效。</p>
<pre class="lang-go" data-nodeid="211792"><code data-language="go">        cgroup := cgroups.NewCgroups()
        <span class="hljs-keyword">defer</span> cgroup.Destroy()
        cpus := context.Int(<span class="hljs-string">"cpus"</span>)
        <span class="hljs-keyword">if</span> cpus != <span class="hljs-number">0</span> {
            cgroup.SetCPULimit(cpus)
        }
        m := context.Int(<span class="hljs-string">"m"</span>)
        <span class="hljs-keyword">if</span> m != <span class="hljs-number">0</span> {
            cgroup.SetMemoryLimit(m)
        }
        cgroup.Apply(cmd.Process.Pid)
</code></pre>
<p data-nodeid="211793">到此，我们成功实现了一个带有资源限制的 gocker 容器。下面进入 gocker 的目录，并且编译一下 gocker：</p>
<pre class="lang-dart" data-nodeid="218325"><code data-language="dart">$ cd gocker
$ git checkout lesson<span class="hljs-number">-18</span>
$ go install
</code></pre>
<p data-nodeid="218326">执行完 go install 后， Golang 会自动帮助我们编译当前项目下的代码，编译后的二进制文件存放在 $GOPATH/bin 目录下，由于我们之前在 $HOME/.bashrc 文件下把 $GOPATH/bin 放入了系统 PATH 中，所以此时你可以直接使用 gocker 命令了。</p>
<h4 data-nodeid="218327">启动带有资源限制的容器</h4>
<p data-nodeid="218328">接下来我们使用 gocker 来启动一个带有 CPU 限制的容器：</p>
<pre class="lang-dart" data-nodeid="220424"><code data-language="dart"># gocker run -it -cpus=<span class="hljs-number">1</span> -rootfs=/tmp/busybox /bin/sh
<span class="hljs-number">2020</span>/<span class="hljs-number">09</span>/<span class="hljs-number">19</span> <span class="hljs-number">23</span>:<span class="hljs-number">46</span>:<span class="hljs-number">27</span> Current path <span class="hljs-keyword">is</span>&nbsp; /tmp/busybox
<span class="hljs-number">2020</span>/<span class="hljs-number">09</span>/<span class="hljs-number">19</span> <span class="hljs-number">23</span>:<span class="hljs-number">46</span>:<span class="hljs-number">27</span> CmdArray <span class="hljs-keyword">is</span>&nbsp; [/bin/sh]
/ #
</code></pre>
<p data-nodeid="220425">然后我们新打开一个命令行窗口，查看一下 cgroups 相关的文件是否被创建：</p>
<pre class="lang-dart" data-nodeid="222507"><code data-language="dart"># cd /sys/fs/cgroup/cpu
# ls -l
总用量 <span class="hljs-number">0</span>
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cgroup.clone_children
--w--w--w-&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cgroup.event_control
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cgroup.procs
-r--r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cgroup.sane_behavior
-r--r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cpuacct.stat
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cpuacct.usage
-r--r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cpuacct.usage_percpu
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cpu.cfs_period_us
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cpu.cfs_quota_us
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cpu.rt_period_us
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cpu.rt_runtime_us
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cpu.shares
-r--r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> cpu.stat
drwxr-xr-x&nbsp; <span class="hljs-number">2</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> gocker
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> notify_on_release
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> release_agent
drwxr-xr-x <span class="hljs-number">70</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">24</span> system.slice
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> tasks
drwxr-xr-x&nbsp; <span class="hljs-number">2</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> user.slice
</code></pre>
<p data-nodeid="222508">可以看到我们启动容器后， gocker 在 cpu 子系统下，已经成功创建 gocker 目录。然后我们查看一下 gocker 目录下的内容：</p>
<pre class="lang-dart" data-nodeid="224578"><code data-language="dart"># ls -l gocker/
总用量 <span class="hljs-number">0</span>
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cgroup.clone_children
--w--w--w- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cgroup.event_control
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cgroup.procs
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cpuacct.stat
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cpuacct.usage
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cpuacct.usage_percpu
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cpu.cfs_period_us
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cpu.cfs_quota_us
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cpu.rt_period_us
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cpu.rt_runtime_us
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cpu.shares
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> cpu.stat
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> notify_on_release
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> <span class="hljs-number">9</span>月&nbsp; <span class="hljs-number">22</span> <span class="hljs-number">20</span>:<span class="hljs-number">48</span> tasks
</code></pre>
<p data-nodeid="224579">可以看到 cgroups 已经帮我们初始化好了 cpu 子系统的文件，然后我们查看一下  cpu.cfs_quota_us 的内容：</p>
<pre class="lang-dart" data-nodeid="226625"><code data-language="dart"># cat gocker/cpu.cfs_quota_us
<span class="hljs-number">100000</span>
</code></pre>
<p data-nodeid="226626">可以看到我们容器的 CPU资源已经被限制为 1 核。下面我们来验证一下 CPU 限制是否生效。<br>
首先我们在容器窗口使用以下命令制造一个死循环，来提升 cpu 使用率：</p>
<pre class="lang-dart" data-nodeid="228650"><code data-language="dart"># <span class="hljs-keyword">while</span> <span class="hljs-keyword">true</span>;<span class="hljs-keyword">do</span> echo;done;
</code></pre>
<p data-nodeid="228651">然后在主机的窗口使用 top 查看一下cpu 使用率：</p>
<pre class="lang-dart" data-nodeid="230661"><code data-language="dart">top - <span class="hljs-number">20</span>:<span class="hljs-number">57</span>:<span class="hljs-number">50</span> up <span class="hljs-number">2</span> days, <span class="hljs-number">23</span>:<span class="hljs-number">23</span>,&nbsp; <span class="hljs-number">2</span> users,&nbsp; load average: <span class="hljs-number">1.08</span>, <span class="hljs-number">0.27</span>, <span class="hljs-number">0.14</span>
Tasks: <span class="hljs-number">113</span> total,&nbsp; &nbsp;<span class="hljs-number">4</span> running, <span class="hljs-number">109</span> sleeping,&nbsp; &nbsp;<span class="hljs-number">0</span> stopped,&nbsp; &nbsp;<span class="hljs-number">0</span> zombie
%Cpu(s): <span class="hljs-number">23.5</span> us, <span class="hljs-number">26.9</span> sy,&nbsp; <span class="hljs-number">0.0</span> ni, <span class="hljs-number">49.2</span> id,&nbsp; <span class="hljs-number">0.0</span> wa,&nbsp; <span class="hljs-number">0.0</span> hi,&nbsp; <span class="hljs-number">0.3</span> si,&nbsp; <span class="hljs-number">0.0</span> st
KiB Mem :&nbsp; <span class="hljs-number">3880512</span> total,&nbsp; <span class="hljs-number">1573052</span> free,&nbsp; &nbsp;<span class="hljs-number">408696</span> used,&nbsp; <span class="hljs-number">1898764</span> buff/cache
KiB Swap:&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> total,&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> free,&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> used.&nbsp; <span class="hljs-number">3141076</span> avail Mem

&nbsp; PID USER&nbsp; &nbsp; &nbsp; PR&nbsp; NI&nbsp; &nbsp; VIRT&nbsp; &nbsp; RES&nbsp; &nbsp; SHR S&nbsp; %CPU %MEM&nbsp; &nbsp; &nbsp;TIME+ COMMAND
<span class="hljs-number">30766</span> root&nbsp; &nbsp; &nbsp; <span class="hljs-number">20</span>&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">1312</span>&nbsp; &nbsp; <span class="hljs-number">260</span>&nbsp; &nbsp; <span class="hljs-number">212</span> R&nbsp; <span class="hljs-number">99.3</span>&nbsp; <span class="hljs-number">0.0</span>&nbsp; &nbsp;<span class="hljs-number">0</span>:<span class="hljs-number">30.90</span> sh
</code></pre>
<p data-nodeid="231163">通过 top 的输出可以看到我们的容器 cpu 使用率被限制到了 100% 以内，即 1 个核。</p>
<p data-nodeid="231164">到此，我们的容器不仅有了 Namespace 隔离，同时也有了 cgroups 的资源限制。</p>

<h3 data-nodeid="230663">结语</h3>
<p data-nodeid="230664">上一课时和本课时，我们一起安装了 golang，并且使用 golang 实现了一个精简版的 Docker，它具有基本的 namespace 隔离，并且还使用 cgroups 对容器进行了资源限制。</p>
<p data-nodeid="230665">这两个课时的关键技术我帮你总结如下。</p>
<ol data-nodeid="230666">
<li data-nodeid="230667">
<p data-nodeid="230668">Linux 的 /proc 目录是一种“文件系统”，它存放于内存中，是一个虚拟的文件系统，/proc 目录存放了当前内核运行状态的一系列特殊的文件，你可以通过这些文件查看当前的进程信息。</p>
</li>
<li data-nodeid="230669">
<p data-nodeid="230670">/proc/self/exe 是一个特殊的连接，执行该文件等同于执行当前程序的二进制文件</p>
</li>
<li data-nodeid="230671">
<p data-nodeid="230672">pivotRoot 是一个系统调用，主要功能是改变当前进程的根目录，它可以把当前进程的根目录移动到我们传递的 rootfs 目录下</p>
</li>
<li data-nodeid="230673">
<p data-nodeid="230674">syscall.Exec 是一个系统调用，这个系统调用可以实现新的进程直接替换正在执行的老的进程，并且复用老进程的 ID。</p>
</li>
</ol>
<p data-nodeid="230675">另外，容器的实现当然离不开 Linux 的 namespace 和 cgroups 这两项关键技术，有了 Linux 的这些关键技术才使得我们的容器可以顺利实现，可以说 Linux 是容器技术的基石。而容器的编写，我们不仅可以使用 Go 语言，也可以使用其他编程语言，甚至只使用 shell 命令也可以实现一个容器。</p>
<p data-nodeid="230676">那么，你可以使用 shell 命令实现一个精简版的 Docker 吗？思考后，不妨试着写一下。</p>
<p data-nodeid="230677">下一课时，我将教你使用 Docker Compose 解决开发环境的依赖。</p>
<p data-nodeid="230678">本课时的源码详见<a href="https://github.com/wilhelmguo/gocker/tree/lesson-18" data-nodeid="230695">这里</a>。</p>

---

### 精选评论

##### **军：
> 老师，我发现源码里限制CPU使用未生效，已提PR给您

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 非常感谢这位同学的 PR，已经合并了。

##### *瑞：
> 都这里评论明显少了，看来懂go的相对java还是少，最近在看 自己动手写docker这本书，关于Linux 内核的一些知识有所欠缺，导致看起来很吃力，现在看老师这两篇源码文章，再结合那本书，原理方面顿时明白了好多，赞！

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 你也很赞！

##### **冬：
> 非常感谢这位老师写的这个系列, 写得很清晰, 非常的深入浅出.看过各种 docker/k8S的书籍,或者课程, 您这边的讲解 把我以前的很多知识点汇聚在一起了.

