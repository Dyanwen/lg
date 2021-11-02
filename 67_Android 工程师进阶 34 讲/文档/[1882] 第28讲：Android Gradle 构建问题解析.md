<p data-nodeid="57102">想必我们做 Android App 开发的应该对 gradle 都不太陌生，因为 Android Studio 的帮助，Android 工程师使用 gradle 的门槛不算太高，基本的配置都大同小异，只要在 Android Studio 默认生成的 build.gradle 中稍加修改，都能满足项目要求。但是深入细致的了解 gradle 的基本知识，还是能够帮助我们更优雅地实现项目的配置工作。有些场景 gradle 甚至能帮助我们完成一些业务上的需要。这节课我们就来了解一下 gradle 那些需要掌握的基本知识。</p>



<h3 data-nodeid="59235" class="">gradle Task</h3>




<p data-nodeid="56024">Task（任务）可以理解为 gradle 的执行单元，gradle 通过执行一个个 Task 来完成整个项目构建工作。</p>
<h4 data-nodeid="61344" class="">自定义 Task</h4>




<p data-nodeid="62378">我们可以在 build.gradle 中使用关键字 task 来自定义一个 Task。比如创建 build.gradle 文件，并添加 task，如图所示：</p>
<p data-nodeid="62379" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/49/Ciqc1F7xxgaAK7DzAAAhdClI9Ok777.png" alt="Drawing 0.png" data-nodeid="62383"></p>


<p data-nodeid="63416">上图中定义了一个简单的 Task A，然后在终端中使用以下命令行执行此 Task，即可看到打印结果，如图所示：</p>
<p data-nodeid="63417" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/49/Ciqc1F7xxg2ASjrQAAA-wRdqLLE574.png" alt="Drawing 1.png" data-nodeid="63421"></p>


<p data-nodeid="64454">从结果中可以看出，打印日志是在 gradle 的配置（Configure）阶段执行的。gradle 的构建生命周期包含 3 部分：初始化阶段、配置阶段、执行阶段。在 task A 中添加 doFirst 闭包，如下所示：</p>
<p data-nodeid="64455" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/55/CgqCHl7xxhSASqSaAABAuGydsVs250.png" alt="Drawing 2.png" data-nodeid="64459"></p>


<p data-nodeid="65492">再次执行 gradle A，打印结果如下所示：</p>
<p data-nodeid="65493" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/49/Ciqc1F7xxhuAZu1aAAB3c1S1xIY247.png" alt="Drawing 3.png" data-nodeid="65497"></p>


<p data-nodeid="56034">gradle 在运行期会执行所有 task 的配置语句，然后执行指定的 Task。</p>
<h4 data-nodeid="56035"><strong data-nodeid="56148">Task 之间可以存在依赖关系</strong></h4>
<p data-nodeid="66530">gradle 中的 Task 可以通过 dependsOn 来指定它依赖另一个 Task，如下所示：</p>
<p data-nodeid="66531" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/49/Ciqc1F7xxiaAIQLqAACWHTcmKWM251.png" alt="Drawing 4.png" data-nodeid="66535"></p>


<p data-nodeid="67568">在 build.gradle 中，我新加了一个 Task B，并通过 dependsOn 关键字指定 task B 依赖于 task A。 在命令行中执行“gradle B”，结果如下所示：</p>
<p data-nodeid="67569" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/55/CgqCHl7xxiyAXQCyAACI5K6PU3A663.png" alt="Drawing 5.png" data-nodeid="67573"></p>


<p data-nodeid="56040">可以看出虽然我们只是执行了 task B，但是因为依赖关系的存在，task A 也会被执行。</p>
<blockquote data-nodeid="56041">
<p data-nodeid="56042">gradle 会在配置 Configure 阶段，确定依赖关系。对于 Android 项目来说即为执行各个 module 下的 build.gradle 文件，这样各个 build.gradle 文件中的 task 的依赖关系就被确认下来了，而这个依赖关系的确定就是在 Configuration 阶段。</p>
</blockquote>
<h4 data-nodeid="56043"><strong data-nodeid="56162">gradle 自定义方法</strong></h4>
<p data-nodeid="68606">我们可以在 build.gradle 中使用 def 关键字，自定义方法，比如以下代码中自定义了 getDate 方法，并在 task 中使用此方法。</p>
<p data-nodeid="68607" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/49/Ciqc1F7xxjOAJ75wAADeC1_69tw793.png" alt="Drawing 6.png" data-nodeid="68611"></p>


<p data-nodeid="69644">通过 gradle 命令执行上述 task，结果如下：</p>
<p data-nodeid="69645" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxjqAcNFQAAI2WxDnREo876.png" alt="Drawing 7.png" data-nodeid="69649"></p>


<h4 data-nodeid="56048"><strong data-nodeid="56174">系统预置 task</strong></h4>
<p data-nodeid="56049">自定义 task 时，还可以使用系统提供的各种显示 task 来完成相应的任务。具体就是使用关键字 type 来指定使用的是哪一个 task。</p>
<p data-nodeid="70682">比如我在当前目录下新建 2 个文件夹：src 和 dst。目录如下：</p>
<p data-nodeid="70683" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/55/CgqCHl7xxkOAcKW5AAA3oZfknVQ144.png" alt="Drawing 8.png" data-nodeid="70687"></p>


<p data-nodeid="71720">然后在 src 中创建文件 Demo.java，代码如下</p>
<p data-nodeid="71721" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxkqAObhEAABdjxnPB1E940.png" alt="Drawing 9.png" data-nodeid="71725"></p>


<p data-nodeid="72758">最后当前路径结构如下：</p>
<p data-nodeid="72759" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/55/CgqCHl7xxlOAZGmoAAA_nRw8Nyc087.png" alt="Drawing 10.png" data-nodeid="72763"></p>


<p data-nodeid="73796">修改 build.gradle，新添加一个 task copy 如下：</p>
<p data-nodeid="73797" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/55/CgqCHl7xxlmAFJTRAAAvcum9iVg859.png" alt="Drawing 11.png" data-nodeid="73801"></p>


<p data-nodeid="74834">然后在命令行中执行“gradle copy”，运行结束后，重新查看当前目录结构如下：</p>
<p data-nodeid="74835" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/55/CgqCHl7xxmCAcAt0AABFeaaAlQc750.png" alt="Drawing 12.png" data-nodeid="74839"></p>


<p data-nodeid="56060">可以看出 Demo.java 被拷贝了一份到 dst 目录中。</p>
<p data-nodeid="75872">除了 Copy 之外，还有很多其他显示的 task 可用，比如我们可以通过自定义 task 实现编译 Demo.java 并将编译后的 .class 输出到某一特定路径，具体实现如下所示：</p>
<p data-nodeid="75873" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/55/CgqCHl7xxmaAIamAAAB21gyJhko466.png" alt="Drawing 13.png" data-nodeid="75877"></p>


<p data-nodeid="56063">解释说明：</p>
<ol data-nodeid="56064">
<li data-nodeid="56065">
<p data-nodeid="56066">通过 type: JavaCompile 指定是编译 Java 类的 task；</p>
</li>
<li data-nodeid="56067">
<p data-nodeid="56068">source 指定需要编译类的文件路径；</p>
</li>
<li data-nodeid="56069">
<p data-nodeid="56070">include 指定需要编译哪一个 Java 类；</p>
</li>
<li data-nodeid="56071">
<p data-nodeid="56072">destinationDir 指定编译之后，生成 .class 文件的保存路径。</p>
</li>
</ol>
<p data-nodeid="76910">最后命令行中执行“gradle compile”，查看目录结构如下：</p>
<p data-nodeid="76911" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxm-AGk5NAABYQ4efSLE744.png" alt="Drawing 14.png" data-nodeid="76915"></p>


<h3 data-nodeid="56075"><strong data-nodeid="56213">gradle project</strong></h3>
<p data-nodeid="77950">在 Android 中每个 module 就对应着一个 project，gradle 在编译时期会为每一个 project 创建一个 Project 对象用来构建项目。这一过程是在初始化阶段，通过解析 settings.gradle 中的配置来创建相应的 Project。</p>
<p data-nodeid="77951" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/55/CgqCHl7xxneADC6uAAAwMY8f2jo857.png" alt="Drawing 15.png" data-nodeid="77955"></p>



<p data-nodeid="78988">上图 settings.gradle 中导入了 3 个 project，但是实际上还会有一个根 project，使用 ./gradlew project 查看，如下所示：</p>
<p data-nodeid="78989" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxoCANfOBAAEhWOUbtnQ479.png" alt="Drawing 16.png" data-nodeid="78993"></p>


<p data-nodeid="80028">我们可以在根 project 中统筹管理所有的子 project，具体在 LagouGradle 路径下的 build.gradle 中进行设置，如下所示：</p>
<p data-nodeid="80029" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/55/CgqCHl7xxoqASIQyAAJT5Xo0vsE094.png" alt="Drawing 17.png" data-nodeid="80033"></p>



<p data-nodeid="81066">这样写的好处是项目中所有 module 的配置都统一写在一个地方，统筹管理。比如经常会在主项目的 build.gradle 中添加包过滤，解决依赖冲突，如下所示：</p>
<p data-nodeid="81067" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxpOAT_XTAAKMtjb8INs789.png" alt="Drawing 18.png" data-nodeid="81071"></p>


<h3 data-nodeid="56084"><strong data-nodeid="56233">buildSrc 统筹依赖管理</strong></h3>
<p data-nodeid="82104">随着项目越来越大，工程中的 module 越来越多，依赖的三方库也越来越多。一般情况下我们会在一个集中的地方统一管理这些三方库的版本。比如像谷歌官方推荐的使用 ext 变量，在根 module 的 build.gradle 中，使用 ext 集中声明各种三方库的版本，如下所示：</p>
<p data-nodeid="82105" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/56/CgqCHl7xxqWAEgffAAGk5hqywVA752.png" alt="Drawing 19.png" data-nodeid="82109"></p>


<p data-nodeid="83142">然后在子 module 中，引用这些版本信息。</p>
<p data-nodeid="83143" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/56/CgqCHl7xxq6AL2vaAAHwPPKyVNo666.png" alt="Drawing 20.png" data-nodeid="83147"></p>


<p data-nodeid="83930">但是这种写法有点小瑕疵：不支持 AS 的自动补充功能，也无法使用代码自动跟踪，因此可以考虑使用 buildSrc。</p>


<p data-nodeid="56090">buildSrc 是 Android 项目中一个比较特殊的 project，在 buildSrc 中可以编写 Groovy 语言，但是现在谷歌越来也推荐使用 Kotlin 来编写编译语句。</p>
<p data-nodeid="84704">先在根路径下创建目录 buildSrc，结构如下：</p>
<p data-nodeid="84705" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxriAUHEZAABrS7D3W5Y817.png" alt="Drawing 21.png" data-nodeid="84709"></p>


<blockquote data-nodeid="56093">
<p data-nodeid="56094"><strong data-nodeid="56252">注意</strong>：这个工程的只能有一个，并且名字必须为 buildSrc。</p>
</blockquote>
<p data-nodeid="86266">创建好之后，在 buildSrc 中创建 build.gradle.kts 文件，并添加 Kotlin 插件。</p>
<p data-nodeid="86267" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxsGAMYgAAABS_wU3BLs527.png" alt="Drawing 22.png" data-nodeid="86271"></p>



<p data-nodeid="87304">编译工程有可能会报错，如下所示：</p>
<p data-nodeid="87305" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxsiAXosXAAG9tqr61q0366.png" alt="Drawing 23.png" data-nodeid="87309"></p>


<p data-nodeid="56099">只要添加 repositories { jcenter() }&nbsp;仓库即可。</p>
<p data-nodeid="88342">接下来在 buildSrc 中创建 src/main/java 目录，并在此目录下创建 Dependencies.kt（名字可随意取）。在 Dependencies.kt 中创建两个 object，分别用来管理工程中的版本信息以及依赖库。</p>
<p data-nodeid="88343" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxtCAVD7QAABV55LwZYk087.png" alt="Drawing 24.png" data-nodeid="88347"></p>


<p data-nodeid="89380">我们可以在 Versions 中添加各种项目中可能会引用到的版本。</p>
<p data-nodeid="89381" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxtiAOo-fAAHpeqvEX5Q157.png" alt="Drawing 25.png" data-nodeid="89385"></p>


<p data-nodeid="90418">然后在 Deps 中引用 Versions 中的变量。</p>
<p data-nodeid="90419" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/56/CgqCHl7xxt-ADJo0AAIfd9-Y62o105.png" alt="Drawing 26.png" data-nodeid="90423"></p>


<p data-nodeid="91456">最后我们就可以在各个 module 中的 build.gradle 中直接使用 Deps 中的变量用来声明依赖，比如在 app module 的 build.gradle 中添加如下依赖。</p>
<p data-nodeid="91457" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4A/Ciqc1F7xxuWAFi1dAACYA6J2oxs814.png" alt="Drawing 27.png" data-nodeid="91461"></p>


<p data-nodeid="93532" class="te-preview-highlight">上图中分别是使用 buildSrc 前后的对比，并且在使用 Deps 的过程中，studio 会给出自动提示，如下：</p>
<p data-nodeid="93533" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/56/CgqCHl7xx4GAZqZQAHlyz8uwUhg260.gif" alt="image.gif" data-nodeid="93537"></p>


<h3 data-nodeid="56110">总结</h3>
<p data-nodeid="56111">这节课主要介绍了 gradle 构建中的 task 和 project。</p>
<p data-nodeid="92494">task 与大部分的开发者开发是最为紧密的，它是 gradle 构建的基本单元。我们每次编译工程时，Android studio 会在控制台打印出执行的 task 名称，类似下图中的格式。</p>
<p data-nodeid="92495" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/4B/Ciqc1F7xx0WAI7WuAAe7RivnMag830.png" alt="Drawing 29.png" data-nodeid="92499"></p>


<p data-nodeid="56114">我们也可以自定义 task 实现相同的构建需求。</p>
<p data-nodeid="56830">project 对应的项目中的 module，每个 module 中包含一个 build.gradle，每个 build.gradle 都会被 gradle 编译成 Project 字节码。我们在 build.gradle 中所写的所有逻辑，实际上最终都会被映射成此 Project 字节码内的实现逻辑。</p>

---

### 精选评论

##### *启：
> buildSrc方法学习了

##### *陈：
> buildSrc学习了，又get到一个新的技能点

##### **飞：
> buildsrc学习了

##### *涛：
> buildSrc学习了

