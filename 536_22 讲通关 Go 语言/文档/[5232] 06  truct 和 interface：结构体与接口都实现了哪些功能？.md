<p data-nodeid="39536">上节课我留了一个思考题：方法是否可以赋值给一个变量？如果可以，要怎么调用它呢？答案是完全可以，方法赋值给变量称为方法表达式，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="39537"><code data-language="go">age:=Age(<span class="hljs-number">25</span>)
<span class="hljs-comment">//方法赋值给变量，方法表达式</span>
sm:=Age.String
<span class="hljs-comment">//通过变量，要传一个接收者进行调用也就是age</span>
sm(age)
</code></pre>
<p data-nodeid="39538">我们知道，方法 String 其实是没有参数的，但是通过方法表达式赋值给变量 sm 后，在调用的时候，必须要传一个接收者，这样 sm 才知道怎么调用。</p>
<blockquote data-nodeid="39539">
<p data-nodeid="39540">小提示：不管方法是否有参数，通过方法表达式调用，第一个参数必须是接收者，然后才是方法自身的参数。</p>
</blockquote>
<p data-nodeid="39541">下面开始我们今天的课程。之前讲到的类型如整型、字符串等只能描述单一的对象，如果是聚合对象，就无法描述了，比如一个人具备的名字、年龄和性别等信息。因为人作为对象是一个聚合对象，要想描述它需要使用这节课要讲的结构体。</p>
<h3 data-nodeid="39542">结构体</h3>
<h4 data-nodeid="39543">结构体定义</h4>
<p data-nodeid="39544">结构体是一种聚合类型，里面可以包含任意类型的值，这些值就是我们定义的结构体的成员，也称为字段。在 Go 语言中，要自定义一个结构体，需要使用 type+struct 关键字组合。</p>
<p data-nodeid="39545">在下面的例子中，我自定义了一个结构体类型，名称为 person，表示一个人。这个 person 结构体有两个字段：name 代表这个人的名字，age 代表这个人的年龄。</p>
<p data-nodeid="39546"><em data-nodeid="39707"><strong data-nodeid="39706">ch06/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="39547"><code data-language="go"><span class="hljs-keyword">type</span> person <span class="hljs-keyword">struct</span> {
    name <span class="hljs-keyword">string</span>
    age <span class="hljs-keyword">uint</span>
}
</code></pre>
<p data-nodeid="39548">在定义结构体时，字段的声明方法和平时声明一个变量是一样的，都是变量名在前，类型在后，只不过在结构体中，变量名称为成员名或字段名。</p>
<p data-nodeid="39549">结构体的成员字段并不是必需的，也可以一个字段都没有，这种结构体成为空结构体。</p>
<p data-nodeid="39550">根据以上信息，我们可以总结出结构体定义的表达式，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="39551"><code data-language="go"><span class="hljs-keyword">type</span> structName <span class="hljs-keyword">struct</span>{
    fieldName typeName
    ....
    ....
}
</code></pre>
<p data-nodeid="39552">其中：</p>
<ul data-nodeid="39553">
<li data-nodeid="39554">
<p data-nodeid="39555">type 和 struct 是 Go 语言的关键字，二者组合就代表要定义一个新的结构体类型。</p>
</li>
<li data-nodeid="39556">
<p data-nodeid="39557">structName 是结构体类型的名字。</p>
</li>
<li data-nodeid="39558">
<p data-nodeid="39559">fieldName 是结构体的字段名，而 typeName 是对应的字段类型。</p>
</li>
<li data-nodeid="39560">
<p data-nodeid="39561">字段可以是零个、一个或者多个。</p>
</li>
</ul>
<blockquote data-nodeid="39562">
<p data-nodeid="39563">小提示：结构体也是一种类型，所以以后自定义的结构体，我会称为某结构体或某类型，两者是一个意思。比如 person 结构体和 person 类型其实是一个意思。</p>
</blockquote>
<p data-nodeid="39564">定义好结构体后就可以使用了，因为它是一个聚合类型，所以比普通的类型可以携带更多数据。</p>
<h4 data-nodeid="39565">结构体声明使用</h4>
<p data-nodeid="39566">结构体类型和普通的字符串、整型一样，也可以使用同样的方式声明和初始化。</p>
<p data-nodeid="39567">在下面的例子中，我声明了一个 person 类型的变量 p，因为没有对变量 p 初始化，所以默认会使用结构体里字段的零值。</p>
<pre class="lang-go" data-nodeid="39568"><code data-language="go"><span class="hljs-keyword">var</span> p person
</code></pre>
<p data-nodeid="39569">当然在声明一个结构体变量的时候，也可以通过结构体字面量的方式初始化，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="39570"><code data-language="go">p:=person{<span class="hljs-string">"飞雪无情"</span>,<span class="hljs-number">30</span>}
</code></pre>
<p data-nodeid="39571">采用简短声明法，同时采用字面量初始化的方式，把结构体变量 p 的 name 初始化为“飞雪无情”，age 初始化为 30，以逗号分隔。</p>
<p data-nodeid="39572">声明了一个结构体变量后就可以使用了，下面我们运行以下代码，验证 name 和 age 的值是否和初始化的一样。</p>
<pre class="lang-go" data-nodeid="39573"><code data-language="go">fmt.Println(p.name,p.age)
</code></pre>
<p data-nodeid="39574">在 Go 语言中，访问一个结构体的字段和调用一个类型的方法一样，都是使用点操作符“.”。</p>
<p data-nodeid="39575">采用字面量初始化结构体时，初始化值的顺序很重要，必须和字段定义的顺序一致。</p>
<p data-nodeid="39576">在 person 这个结构体中，第一个字段是 string 类型的 name，第二个字段是 uint 类型的 age，所以在初始化的时候，初始化值的类型顺序必须一一对应，才能编译通过。也就是说，在示例 {"飞雪无情",30} 中，表示 name 的字符串飞雪无情必须在前，表示年龄的数字 30 必须在后。</p>
<p data-nodeid="39577">那么是否可以不按照顺序初始化呢？<strong data-nodeid="39736">当然可以，只不过需要指出字段名称</strong>，如下所示：</p>
<pre class="lang-go" data-nodeid="39578"><code data-language="go">p:=person{age:<span class="hljs-number">30</span>,name:<span class="hljs-string">"飞雪无情"</span>}
</code></pre>
<p data-nodeid="39579">其中，第一位我放了整型的 age，也可以编译通过，因为采用了明确的 field:value 方式进行指定，这样 Go 语言编译器会清晰地知道你要初始化哪个字段的值。</p>
<p data-nodeid="39580">有没有发现，这种方式和 map 类型的初始化很像，都是采用冒号分隔。Go 语言尽可能地重用操作，不发明新的表达式，便于我们记忆和使用。</p>
<p data-nodeid="39581">当然你也可以只初始化字段 age，字段 name 使用默认的零值，如下面的代码所示，仍然可以编译通过。</p>
<pre class="lang-go" data-nodeid="39582"><code data-language="go">p:=person{age:<span class="hljs-number">30</span>}
</code></pre>
<h4 data-nodeid="39583">字段结构体</h4>
<p data-nodeid="39584">结构体的字段可以是任意类型，也包括自定义的结构体类型，比如下面的代码：</p>
<p data-nodeid="39585"><em data-nodeid="39746"><strong data-nodeid="39745">ch06/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="39586"><code data-language="go"><span class="hljs-keyword">type</span> person <span class="hljs-keyword">struct</span> {
    name <span class="hljs-keyword">string</span>
    age <span class="hljs-keyword">uint</span>
    addr address
}
<span class="hljs-keyword">type</span> address <span class="hljs-keyword">struct</span> {
    province <span class="hljs-keyword">string</span>
    city <span class="hljs-keyword">string</span>
}
</code></pre>
<p data-nodeid="39587">在这个示例中，我定义了两个结构体：person 表示人，address 表示地址。在结构体 person 中，有一个 address 类型的字段 addr，这就是自定义的结构体。</p>
<p data-nodeid="39588">通过这种方式，用代码描述现实中的实体会更匹配，复用程度也更高。对于嵌套结构体字段的结构体，其初始化和正常的结构体大同小异，只需要根据字段对应的类型初始化即可，如下面的代码所示：</p>
<p data-nodeid="39589"><em data-nodeid="39753"><strong data-nodeid="39752">ch06/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="39590"><code data-language="go">    p:=person{
        age:<span class="hljs-number">30</span>,
        name:<span class="hljs-string">"飞雪无情"</span>,
        addr:address{
            province: <span class="hljs-string">"北京"</span>,
            city:     <span class="hljs-string">"北京"</span>,
        },
    }
</code></pre>
<p data-nodeid="39591">如果需要访问结构体最里层的 province 字段的值，同样也可以使用点操作符，只不过需要使用两个点，如下面的代码所示：</p>
<p data-nodeid="39592"><em data-nodeid="39759"><strong data-nodeid="39758">ch06/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="39593"><code data-language="go">fmt.Println(p.addr.province)
</code></pre>
<p data-nodeid="39594">第一个点获取 addr，第二个点获取 addr 的 province。</p>
<h3 data-nodeid="39595">接口</h3>
<h4 data-nodeid="39596">接口的定义</h4>
<p data-nodeid="39597">接口是和调用方的一种约定，它是一个高度抽象的类型，不用和具体的实现细节绑定在一起。接口要做的是定义好约定，告诉调用方自己可以做什么，但不用知道它的内部实现，这和我们见到的具体的类型如 int、map、slice 等不一样。</p>
<p data-nodeid="39598">接口的定义和结构体稍微有些差别，虽然都以 type 关键字开始，但接口的关键字是 interface，表示自定义的类型是一个接口。也就是说 Stringer 是一个接口，它有一个方法 String() string，整体如下面的代码所示：</p>
<p data-nodeid="39599"><em data-nodeid="39769"><strong data-nodeid="39768">src/fmt/print.go</strong></em></p>
<pre class="lang-go" data-nodeid="39600"><code data-language="go"><span class="hljs-keyword">type</span> Stringer <span class="hljs-keyword">interface</span> {
    String() <span class="hljs-keyword">string</span>
}
</code></pre>
<blockquote data-nodeid="39601">
<p data-nodeid="39602">提示：Stringer 是 Go SDK 的一个接口，属于 fmt 包。</p>
</blockquote>
<p data-nodeid="39603">针对 Stringer 接口来说，它会告诉调用者可以通过它的 String() 方法获取一个字符串，这就是接口的约定。至于这个字符串怎么获得的，长什么样，接口不关心，调用者也不用关心，因为这些是由接口实现者来做的。</p>
<h4 data-nodeid="39604">接口的实现</h4>
<p data-nodeid="39605">接口的实现者必须是一个具体的类型，继续以 person 结构体为例，让它来实现 Stringer 接口，如下代码所示：</p>
<p data-nodeid="39606"><em data-nodeid="39778"><strong data-nodeid="39777">ch06/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="39607"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(p person)</span> <span class="hljs-title">String</span><span class="hljs-params">()</span>  <span class="hljs-title">string</span></span>{
    <span class="hljs-keyword">return</span> fmt.Sprintf(<span class="hljs-string">"the name is %s,age is %d"</span>,p.name,p.age)
}
</code></pre>
<p data-nodeid="39608">给结构体类型 person 定义一个方法，这个方法和接口里方法的签名（名称、参数和返回值）一样，这样结构体 person 就实现了 Stringer 接口。</p>
<blockquote data-nodeid="39609">
<p data-nodeid="39610">注意：如果一个接口有多个方法，那么需要实现接口的每个方法才算是实现了这个接口。</p>
</blockquote>
<p data-nodeid="39611">实现了 Stringer 接口后就可以使用了。首先我先来定义一个可以打印 Stringer 接口的函数，如下所示：</p>
<p data-nodeid="39612"><em data-nodeid="39786"><strong data-nodeid="39785">ch06/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="39613"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">printString</span><span class="hljs-params">(s fmt.Stringer)</span></span>{
    fmt.Println(s.String())
}
</code></pre>
<p data-nodeid="39614">这个被定义的函数 printString，它接收一个 Stringer 接口类型的参数，然后打印出 Stringer 接口的 String 方法返回的字符串。</p>
<p data-nodeid="39615">printString 这个函数的优势就在于它是面向接口编程的，只要一个类型实现了 Stringer 接口，都可以打印出对应的字符串，而不用管具体的类型实现。</p>
<p data-nodeid="39616">因为 person 实现了 Stringer 接口，所以变量 p 可以作为函数 printString 的参数，可以用如下方式打印：</p>
<pre class="lang-go" data-nodeid="39617"><code data-language="go">printString(p)
</code></pre>
<p data-nodeid="39618">结果为：</p>
<pre class="lang-shell" data-nodeid="39619"><code data-language="shell">the name is 飞雪无情,age is 30
</code></pre>
<p data-nodeid="39620">现在让结构体 address 也实现 Stringer 接口，如下面的代码所示：</p>
<p data-nodeid="39621"><em data-nodeid="39796"><strong data-nodeid="39795">ch06/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="39622"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(addr address)</span> <span class="hljs-title">String</span><span class="hljs-params">()</span>  <span class="hljs-title">string</span></span>{
    <span class="hljs-keyword">return</span> fmt.Sprintf(<span class="hljs-string">"the addr is %s%s"</span>,addr.province,addr.city)
}
</code></pre>
<p data-nodeid="39623">因为结构体 address 也实现了 Stringer 接口，所以 printString 函数不用做任何改变，可以直接被使用，打印出地址，如下所示：</p>
<pre class="lang-go" data-nodeid="39624"><code data-language="go">printString(p.addr)
<span class="hljs-comment">//输出：the addr is 北京北京</span>
</code></pre>
<p data-nodeid="39625">这就是面向接口的好处，只要定义和调用双方满足约定，就可以使用，而不用管具体实现。接口的实现者也可以更好的升级重构，而不会有任何影响，因为接口约定没有变。</p>
<h4 data-nodeid="39626">值接收者和指针接收者</h4>
<p data-nodeid="39627">我们已经知道，如果要实现一个接口，必须实现这个接口提供的所有方法，而且在上节课讲解方法的时候，我们也知道定义一个方法，有值类型接收者和指针类型接收者两种。二者都可以调用方法，因为 Go 语言编译器自动做了转换，所以值类型接收者和指针类型接收者是等价的。但是在接口的实现中，值类型接收者和指针类型接收者不一样，下面我会详细分析二者的区别。</p>
<p data-nodeid="39628">在上一小节中，已经验证了结构体类型实现了 Stringer 接口，那么结构体对应的指针是否也实现了该接口呢？我通过下面这个代码进行测试：</p>
<pre class="lang-go" data-nodeid="39629"><code data-language="go">printString(&amp;p)
</code></pre>
<p data-nodeid="39630">测试后会发现，把变量 p 的指针作为实参传给 printString 函数也是可以的，编译运行都正常。这就证明了<strong data-nodeid="39807">以值类型接收者实现接口的时候，不管是类型本身，还是该类型的指针类型，都实现了该接口</strong>。</p>
<p data-nodeid="39631">示例中值接收者（p person）实现了 Stringer 接口，那么类型 person 和它的指针类型*person就都实现了 Stringer 接口。</p>
<p data-nodeid="39632">现在，我把接收者改成指针类型，如下代码所示：</p>
<pre class="lang-go" data-nodeid="39633"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(p *person)</span> <span class="hljs-title">String</span><span class="hljs-params">()</span>  <span class="hljs-title">string</span></span>{
    <span class="hljs-keyword">return</span> fmt.Sprintf(<span class="hljs-string">"the name is %s,age is %d"</span>,p.name,p.age)
}
</code></pre>
<p data-nodeid="39634">修改成指针类型接收者后会发现，示例中这行 printString(p) 代码编译不通过，提示如下错误：</p>
<pre class="lang-shell" data-nodeid="39635"><code data-language="shell">./main.go:17:13: cannot use p (type person) as type fmt.Stringer in argument to printString:
    person does not implement fmt.Stringer (String method has pointer receiver)
</code></pre>
<p data-nodeid="39636">意思就是类型 person 没有实现 Stringer 接口。这就证明了<strong data-nodeid="39817">以指针类型接收者实现接口的时候，只有对应的指针类型才被认为实现了该接口。</strong></p>
<p data-nodeid="39637">我用如下表格为你总结这两种接收者类型的接口实现规则：</p>
<p data-nodeid="39638"><img alt="Drawing 0.png" src="https://s0.lgstatic.com/i/image/M00/6E/59/Ciqc1F-yPMSAZ4k7AABU_GW4VxE080.png" data-nodeid="39821"></p>
<p data-nodeid="39639">可以这样解读：</p>
<ul data-nodeid="39640">
<li data-nodeid="39641">
<p data-nodeid="39642">当值类型作为接收者时，person 类型和*person类型都实现了该接口。</p>
</li>
<li data-nodeid="39643">
<p data-nodeid="39644">当指针类型作为接收者时，只有*person类型实现了该接口。</p>
</li>
</ul>
<p data-nodeid="39645">可以发现，实现接口的类型都有*person，这也表明指针类型比较万能，不管哪一种接收者，它都能实现该接口。</p>
<h3 data-nodeid="39646">工厂函数</h3>
<p data-nodeid="39647">工厂函数一般用于创建自定义的结构体，便于使用者调用，我们还是以 person 类型为例，用如下代码进行定义：</p>
<pre class="lang-go" data-nodeid="39648"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">NewPerson</span><span class="hljs-params">(name <span class="hljs-keyword">string</span>)</span> *<span class="hljs-title">person</span></span> {
    <span class="hljs-keyword">return</span> &amp;person{name:name}
}
</code></pre>
<p data-nodeid="39649">我定义了一个工厂函数 NewPerson，它接收一个 string 类型的参数，用于表示这个人的名字，同时返回一个*person。</p>
<p data-nodeid="39650">通过工厂函数创建自定义结构体的方式，可以让调用者不用太关注结构体内部的字段，只需要给工厂函数传参就可以了。</p>
<p data-nodeid="39651">用下面的代码，即可创建一个*person 类型的变量 p1：</p>
<pre class="lang-go" data-nodeid="39652"><code data-language="go">p1:=NewPerson(<span class="hljs-string">"张三"</span>)
</code></pre>
<p data-nodeid="39653">工厂函数也可以用来创建一个接口，它的好处就是可以隐藏内部具体类型的实现，让调用者只需关注接口的使用即可。</p>
<p data-nodeid="39654">现在我以 errors.New 这个 Go 语言自带的工厂函数为例，演示如何通过工厂函数创建一个接口，并隐藏其内部实现，如下代码所示：</p>
<p data-nodeid="39655"><em data-nodeid="39847"><strong data-nodeid="39846">errors/errors.go</strong></em></p>
<pre class="lang-go" data-nodeid="39656"><code data-language="go"><span class="hljs-comment">//工厂函数，返回一个error接口，其实具体实现是*errorString</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">New</span><span class="hljs-params">(text <span class="hljs-keyword">string</span>)</span> <span class="hljs-title">error</span></span> {
    <span class="hljs-keyword">return</span> &amp;errorString{text}
}
<span class="hljs-comment">//结构体，内部一个字段s，存储错误信息</span>
<span class="hljs-keyword">type</span> errorString <span class="hljs-keyword">struct</span> {
    s <span class="hljs-keyword">string</span>
}
<span class="hljs-comment">//用于实现error接口</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(e *errorString)</span> <span class="hljs-title">Error</span><span class="hljs-params">()</span> <span class="hljs-title">string</span></span> {
    <span class="hljs-keyword">return</span> e.s
}
</code></pre>
<p data-nodeid="39657">其中，errorString 是一个结构体类型，它实现了 error 接口，所以可以通过 New 工厂函数，创建一个 *errorString 类型，通过接口 error 返回。</p>
<p data-nodeid="39658">这就是面向接口的编程，假设重构代码，哪怕换一个其他结构体实现 error 接口，对调用者也没有影响，因为接口没变。</p>
<h3 data-nodeid="39659">继承和组合</h3>
<p data-nodeid="39660">在 Go 语言中没有继承的概念，所以结构、接口之间也没有父子关系，Go 语言提倡的是组合，利用组合达到代码复用的目的，这也更灵活。</p>
<p data-nodeid="39661">我同样以 Go 语言 io 标准包自带的接口为例，讲解类型的组合（也可以称之为嵌套），如下代码所示：</p>
<pre class="lang-go" data-nodeid="39662"><code data-language="go"><span class="hljs-keyword">type</span> Reader <span class="hljs-keyword">interface</span> {
    Read(p []<span class="hljs-keyword">byte</span>) (n <span class="hljs-keyword">int</span>, err error)
}
<span class="hljs-keyword">type</span> Writer <span class="hljs-keyword">interface</span> {
    Write(p []<span class="hljs-keyword">byte</span>) (n <span class="hljs-keyword">int</span>, err error)
}
<span class="hljs-comment">//ReadWriter是Reader和Writer的组合</span>
<span class="hljs-keyword">type</span> ReadWriter <span class="hljs-keyword">interface</span> {
    Reader
    Writer
}
</code></pre>
<p data-nodeid="39663">ReadWriter 接口就是 Reader 和 Writer 的组合，组合后，ReadWriter 接口具有 Reader 和 Writer 中的所有方法，这样新接口 ReadWriter 就不用定义自己的方法了，组合 Reader 和 Writer 的就可以了。</p>
<p data-nodeid="39664">不止接口可以组合，结构体也可以组合，现在把 address 结构体组合到结构体 person 中，而不是当成一个字段，如下所示：</p>
<p data-nodeid="39665"><em data-nodeid="39861"><strong data-nodeid="39860">ch06/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="39666"><code data-language="go"><span class="hljs-keyword">type</span> person <span class="hljs-keyword">struct</span> {
    name <span class="hljs-keyword">string</span>
    age <span class="hljs-keyword">uint</span>
    address
}
</code></pre>
<p data-nodeid="39667">直接把结构体类型放进来，就是组合，不需要字段名。组合后，被组合的 address 称为内部类型，person 称为外部类型。修改了 person 结构体后，声明和使用也需要一起修改，如下所示：</p>
<pre class="lang-go" data-nodeid="39668"><code data-language="go">p:=person{
        age:<span class="hljs-number">30</span>,
        name:<span class="hljs-string">"飞雪无情"</span>,
        address:address{
            province: <span class="hljs-string">"北京"</span>,
            city:     <span class="hljs-string">"北京"</span>,
        },
    }
<span class="hljs-comment">//像使用自己的字段一样，直接使用</span>
fmt.Println(p.province)
</code></pre>
<p data-nodeid="39669">因为 person 组合了 address，所以 address 的字段就像 person 自己的一样，可以直接使用。</p>
<p data-nodeid="39670">类型组合后，外部类型不仅可以使用内部类型的字段，也可以使用内部类型的方法，就像使用自己的方法一样。如果外部类型定义了和内部类型同样的方法，那么外部类型的会覆盖内部类型，这就是方法的覆写。关于方法的覆写，这里不再进行举例，你可以自己试一下。</p>
<blockquote data-nodeid="39671">
<p data-nodeid="39672">小提示：方法覆写不会影响内部类型的方法实现。</p>
</blockquote>
<h3 data-nodeid="39673">类型断言</h3>
<p data-nodeid="39674">有了接口和实现接口的类型，就会有类型断言。类型断言用来判断一个接口的值是否是实现该接口的某个具体类型。</p>
<p data-nodeid="39675">还是以我们上面小节的示例演示，我们先来回忆一下它们，如下所示：</p>
<pre class="lang-go" data-nodeid="39676"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(p *person)</span> <span class="hljs-title">String</span><span class="hljs-params">()</span>  <span class="hljs-title">string</span></span>{
    <span class="hljs-keyword">return</span> fmt.Sprintf(<span class="hljs-string">"the name is %s,age is %d"</span>,p.name,p.age)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(addr address)</span> <span class="hljs-title">String</span><span class="hljs-params">()</span>  <span class="hljs-title">string</span></span>{
    <span class="hljs-keyword">return</span> fmt.Sprintf(<span class="hljs-string">"the addr is %s%s"</span>,addr.province,addr.city)
}
</code></pre>
<p data-nodeid="39677">可以看到，*person 和 address 都实现了接口 Stringer，然后我通过下面的示例讲解类型断言：</p>
<pre class="lang-go" data-nodeid="39678"><code data-language="go">    <span class="hljs-keyword">var</span> s fmt.Stringer
    s = p1
    p2:=s.(*person)
    fmt.Println(p2)
</code></pre>
<p data-nodeid="39679">如上所示，接口变量 s 称为接口 fmt.Stringer 的值，它被 p1 赋值。然后使用类型断言表达式 s.(<em data-nodeid="39879">person)，尝试返回一个 p2。如果接口的值 s 是一个</em>person，那么类型断言正确，可以正常返回 p2。如果接口的值 s 不是一个 *person，那么在运行时就会抛出异常，程序终止运行。</p>
<blockquote data-nodeid="39680">
<p data-nodeid="39681">小提示：这里返回的 p2 已经是 *person 类型了，也就是在类型断言的时候，同时完成了类型转换。</p>
</blockquote>
<p data-nodeid="39682">在上面的示例中，因为 s 的确是一个 *person，所以不会异常，可以正常返回 p2。但是如果我再添加如下代码，对 s 进行 address 类型断言，就会出现一些问题：</p>
<pre class="lang-go" data-nodeid="39683"><code data-language="go">    a:=s.(address)
    fmt.Println(a)
</code></pre>
<p data-nodeid="39684">这个代码在编译的时候不会有问题，因为 address 实现了接口 Stringer，但是在运行的时候，会抛出如下异常信息：</p>
<pre class="lang-go" data-nodeid="39685"><code data-language="go"><span class="hljs-built_in">panic</span>: <span class="hljs-keyword">interface</span> conversion: fmt.Stringer is *main.person, not main.address
</code></pre>
<p data-nodeid="39686">这显然不符合我们的初衷，我们本来想判断一个接口的值是否是某个具体类型，但不能因为判断失败就导致程序异常。考虑到这点，Go 语言为我们提供了类型断言的多值返回，如下所示：</p>
<pre class="lang-go" data-nodeid="39687"><code data-language="go">    a,ok:=s.(address)
    <span class="hljs-keyword">if</span> ok {
        fmt.Println(a)
    }<span class="hljs-keyword">else</span> {
        fmt.Println(<span class="hljs-string">"s不是一个address"</span>)
    }
</code></pre>
<p data-nodeid="39688">类型断言返回的第二个值“ok”就是断言是否成功的标志，如果为 true 则成功，否则失败。</p>
<h3 data-nodeid="39689">总结</h3>
<p data-nodeid="39690">这节课虽然只讲了结构体和接口，但是所涉及的知识点很多，整节课比较长，希望你可以耐心地学完。</p>
<p data-nodeid="39691">结构体是对现实世界的描述，接口是对某一类行为的规范和抽象。通过它们，我们可以实现代码的抽象和复用，同时可以面向接口编程，把具体实现细节隐藏起来，让写出来的代码更灵活，适应能力也更强。</p>
<p data-nodeid="39692"><img alt="Drawing 1.png" src="https://s0.lgstatic.com/i/image/M00/6E/65/CgqCHl-yPTiAWp5CAAXli5Grm8o875.png" data-nodeid="39894"></p>
<p data-nodeid="39693">这节课的<strong data-nodeid="39900">练习题是</strong>：自己练习实现有两个方法的接口。</p>
<p class="te-preview-highlight" data-nodeid="39694">下节课我将会为你继续讲解 Go 语言对错误、异常的处理，记得来听课！</p>

---

### 精选评论

##### *涛：
> 错误">p2:=s.(*person) 应该是p2:=s.(person) 去掉*

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 再仔细看下文章，只有指针类型的person才实现了fmt.Stringer接口。

##### **聪：
> //工厂函数，返回一个error接口不是很懂这句话和这部分的内容。errorString是结构体，为什么说返回一个error接口？不是只有interface才叫接口吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为函数声明的返回类型是一个接口。

##### **龙：
> go中没有继承的话，像一些行为接口的通用实现一般要如何去写呢？例如Java的Collection接口中包含size()和isEmpty()方法，Java直接在抽象类AbstractCollection中完成了对isEmpty()方法的实现，使的其子类无需再去实现isEmpty()方法。go中需要如何满足这种情况呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 嵌套，直接嵌套一个类型。

