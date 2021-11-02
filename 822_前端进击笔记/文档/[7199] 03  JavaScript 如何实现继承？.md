<p data-nodeid="1721" class="">JavaScript 在编程语言界是个异类，它和其他编程语言很不一样，JavaScript 可以在运行的时候动态地改变某个变量的类型。</p>
<p data-nodeid="1722">比如你永远也没法想到像<code data-backticks="1" data-nodeid="1849">isTimeout</code>这样一个变量可以存在多少种类型，除了布尔值<code data-backticks="1" data-nodeid="1851">true</code>和<code data-backticks="1" data-nodeid="1853">false</code>，它还可能是<code data-backticks="1" data-nodeid="1855">undefined</code>、<code data-backticks="1" data-nodeid="1857">1</code>和<code data-backticks="1" data-nodeid="1859">0</code>、一个时间戳，甚至一个对象。</p>
<p data-nodeid="1723">又或者你的代码跑异常了，打开浏览器开始断点，发现<code data-backticks="1" data-nodeid="1862">InfoList</code>这个变量第一次被赋值的时候是个数组<code data-backticks="1" data-nodeid="1864">[{name: 'test1', value: '11'}, {name: 'test2', value: '22'}]</code>，过了一会竟然变成了一个对象<code data-backticks="1" data-nodeid="1866">{test1:'11', test2: '22'}</code></p>
<p data-nodeid="1724">除了变量可以在运行时被赋值为任何类型以外，JavaScript 中也能实现继承，但它不像 Java、C++、C# 这些编程语言一样基于类来实现继承，而是基于原型进行继承。</p>
<p data-nodeid="1725">这是因为 JavaScript 中有个特殊的存在：对象。每个对象还都拥有一个原型对象，并可以从中继承方法和属性。</p>
<p data-nodeid="1726">提到对象和原型，你曾经是否有过这些疑惑：</p>
<ol data-nodeid="1727">
<li data-nodeid="1728">
<p data-nodeid="1729">JavaScript 的函数怎么也是个对象？</p>
</li>
<li data-nodeid="1730">
<p data-nodeid="1731"><code data-backticks="1" data-nodeid="1871">__proto__</code>和<code data-backticks="1" data-nodeid="1873">prototype</code>到底是啥关系？</p>
</li>
<li data-nodeid="1732">
<p data-nodeid="1733">JavaScript 中对象是怎么实现继承的？</p>
</li>
<li data-nodeid="1734">
<p data-nodeid="1735">JavaScript 是怎么访问对象的方法和属性的？</p>
</li>
</ol>
<p data-nodeid="1736">下面我们一起结合问题，来探讨下 JavaScript 对象和继承。</p>
<h3 data-nodeid="1737">原型对象和对象是什么关系</h3>
<p data-nodeid="1738">在 JavaScript 中，对象由一组或多组的属性和值组成：</p>
<pre class="lang-java" data-nodeid="1739"><code data-language="java">{
  key1: value1,
  key2: value2,
  key3: value3,
}
</code></pre>
<p data-nodeid="1740">在 JavaScript 中，对象的用途很是广泛，因为它的值既可以是原始类型（<code data-backticks="1" data-nodeid="1881">number</code>、<code data-backticks="1" data-nodeid="1883">string</code>、<code data-backticks="1" data-nodeid="1885">boolean</code>、<code data-backticks="1" data-nodeid="1887">null</code>、<code data-backticks="1" data-nodeid="1889">undefined</code>、<code data-backticks="1" data-nodeid="1891">bigint</code>和<code data-backticks="1" data-nodeid="1893">symbol</code>），还可以是对象和函数。</p>
<p data-nodeid="1741">不管是对象，还是函数和数组，它们都是<code data-backticks="1" data-nodeid="1896">Object</code>的实例，也就是说在 JavaScript 中，除了原始类型以外，其余都是对象。</p>
<p data-nodeid="1742">这也就解答了疑惑 1：JavaScript 的函数怎么也是个对象？</p>
<p data-nodeid="1743">在 JavaScript 中，函数也是一种特殊的对象，它同样拥有属性和值。所有的函数会有一个特别的属性<code data-backticks="1" data-nodeid="1900">prototype</code>，该属性的值是一个对象，这个对象便是我们常说的“原型对象”。</p>
<p data-nodeid="1744">我们可以在控制台打印一下这个属性：</p>
<pre class="lang-java" data-nodeid="1745"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">Person</span><span class="hljs-params">(name)</span> </span>{
  <span class="hljs-keyword">this</span>.name = name;
}
console.log(Person.prototype);
</code></pre>
<p data-nodeid="1746">打印结果显示为：</p>
<p data-nodeid="1747"><img src="https://s0.lgstatic.com/i/image6/M01/34/06/CioPOWBwCzyAM-CAAAAKDg-SVug894.png" alt="Drawing 0.png" data-nodeid="1906"></p>
<p data-nodeid="1748">可以看到，该原型对象有两个属性：<code data-backticks="1" data-nodeid="1908">constructor</code>和<code data-backticks="1" data-nodeid="1910">__proto__</code>。</p>
<p data-nodeid="1749">到这里，我们仿佛看到疑惑 “2：<code data-backticks="1" data-nodeid="1913">__proto__</code>和<code data-backticks="1" data-nodeid="1915">prototype</code>到底是啥关系？”的答案要出现了。在 JavaScript 中，<code data-backticks="1" data-nodeid="1917">__proto__</code>属性指向对象的原型对象，对于函数来说，它的原型对象便是<code data-backticks="1" data-nodeid="1919">prototype</code>。函数的原型对象<code data-backticks="1" data-nodeid="1921">prototype</code>有以下特点：</p>
<ul data-nodeid="1750">
<li data-nodeid="1751">
<p data-nodeid="1752">默认情况下，所有函数的原型对象（<code data-backticks="1" data-nodeid="1924">prototype</code>）都拥有<code data-backticks="1" data-nodeid="1926">constructor</code>属性，该属性指向与之关联的构造函数，在这里构造函数便是<code data-backticks="1" data-nodeid="1928">Person</code>函数；</p>
</li>
<li data-nodeid="1753">
<p data-nodeid="1754"><code data-backticks="1" data-nodeid="1930">Person</code>函数的原型对象（<code data-backticks="1" data-nodeid="1932">prototype</code>）同样拥有自己的原型对象，用<code data-backticks="1" data-nodeid="1934">__proto__</code>属性表示。前面说过，函数是<code data-backticks="1" data-nodeid="1936">Object</code>的实例，因此<code data-backticks="1" data-nodeid="1938">Person.prototype</code>的原型对象为<code data-backticks="1" data-nodeid="1940">Object.prototype。</code></p>
</li>
</ul>
<p data-nodeid="1755">我们可以用这样一张图来描述<code data-backticks="1" data-nodeid="1942">prototype</code>、<code data-backticks="1" data-nodeid="1944">__proto__</code>和<code data-backticks="1" data-nodeid="1946">constructor</code>三个属性的关系：</p>
<p data-nodeid="1756"><img src="https://s0.lgstatic.com/i/image6/M00/39/C6/Cgp9HWB87hmAPbFxAACJvyE_nJI526.png" alt="图片1.png" data-nodeid="1950"></p>
<p data-nodeid="1757">从这个图中，我们可以找到这样的关系：</p>
<ul data-nodeid="1758">
<li data-nodeid="1759">
<p data-nodeid="1760">在 JavaScript 中，<code data-backticks="1" data-nodeid="1953">__proto__</code>属性指向对象的原型对象；</p>
</li>
<li data-nodeid="1761">
<p data-nodeid="1762">对于函数来说，每个函数都有一个<code data-backticks="1" data-nodeid="1956">prototype</code>属性，该属性为该函数的原型对象。</p>
</li>
</ul>
<p data-nodeid="1763">这是否就是疑惑 2 的完整答案呢？并不全是，在 JavaScript 中还可以通过<code data-backticks="1" data-nodeid="1959">prototype</code>和<code data-backticks="1" data-nodeid="1961">__proto__</code>实现继承。</p>
<h3 data-nodeid="1764">使用 prototype 和 <strong data-nodeid="1968">proto</strong> 实现继承</h3>
<p data-nodeid="1765">前面我们说过，对象之所以使用广泛，是因为对象的属性值可以为任意类型。因此，属性的值同样可以为另外一个对象，这意味着 JavaScript 可以这么做：通过将对象 A 的<code data-backticks="1" data-nodeid="1970">__proto__</code>属性赋值为对象 B，即<code data-backticks="1" data-nodeid="1972">A.__proto__ = B</code>，此时使用<code data-backticks="1" data-nodeid="1974">A.__proto__</code>便可以访问 B 的属性和方法。</p>
<p data-nodeid="1766">这样，JavaScript 可以在两个对象之间创建一个关联，使得一个对象可以访问另一个对象的属性和方法，从而实现了继承，此时疑惑 “3. JavaScript 中对象是怎么实现继承的？”解答完毕。</p>
<p data-nodeid="1767">那么，JavaScript 又是怎样使用<code data-backticks="1" data-nodeid="1978">prototype</code>和<code data-backticks="1" data-nodeid="1980">__proto__</code>实现继承的呢？</p>
<p data-nodeid="1768">继续以<code data-backticks="1" data-nodeid="1983">Person</code>为例，当我们使用<code data-backticks="1" data-nodeid="1985">new Person()</code>创建对象时，JavaScript 就会创建构造函数<code data-backticks="1" data-nodeid="1987">Person</code>的实例，比如这里我们创建了一个叫“Lily”的<code data-backticks="1" data-nodeid="1989">Person</code>：</p>
<pre class="lang-java" data-nodeid="1769"><code data-language="java"><span class="hljs-keyword">var</span> lily = <span class="hljs-keyword">new</span> Person(<span class="hljs-string">"Lily"</span>);
</code></pre>
<p data-nodeid="1770">上述这段代码在运行时，JavaScript 引擎通过将<code data-backticks="1" data-nodeid="1992">Person</code>的原型对象<code data-backticks="1" data-nodeid="1994">prototype</code>赋值给实例对象<code data-backticks="1" data-nodeid="1996">lily</code>的<code data-backticks="1" data-nodeid="1998">__proto__</code>属性，实现了<code data-backticks="1" data-nodeid="2000">lily</code>对<code data-backticks="1" data-nodeid="2002">Person</code>的继承，即执行了以下代码：</p>
<pre class="lang-java" data-nodeid="1771"><code data-language="java"><span class="hljs-comment">// 实际上 JavaScript 引擎执行了以下代码</span>
<span class="hljs-keyword">var</span> lily = {};
lily.__proto__ = Person.prototype;
Person.call(lily, <span class="hljs-string">"Lily"</span>);
</code></pre>
<p data-nodeid="1772">我们来打印一下<code data-backticks="1" data-nodeid="2005">lily</code>实例：</p>
<p data-nodeid="1773"><img src="https://s0.lgstatic.com/i/image6/M00/33/FE/Cgp9HWBwC56AVE8iAAAQagv5qXA279.png" alt="Drawing 3.png" data-nodeid="2009"></p>
<p data-nodeid="1774">可以看到，<code data-backticks="1" data-nodeid="2011">lily</code>作为<code data-backticks="1" data-nodeid="2013">Person</code>的实例对象，它的<code data-backticks="1" data-nodeid="2015">__proto__</code>指向了<code data-backticks="1" data-nodeid="2017">Person</code>的原型对象，即<code data-backticks="1" data-nodeid="2019">Person.prototype</code>。</p>
<p data-nodeid="1775">这时，我们再补充下上图中的关系：</p>
<p data-nodeid="1776"><img src="https://s0.lgstatic.com/i/image6/M00/39/CF/CioPOWB87iuAaqLIAADOJoaQI4k669.png" alt="图片2.png" data-nodeid="2024"></p>
<p data-nodeid="1777">从这幅图中，我们可以清晰地看到构造函数和<code data-backticks="1" data-nodeid="2026">constructor</code>属性、原型对象（<code data-backticks="1" data-nodeid="2028">prototype</code>）和<code data-backticks="1" data-nodeid="2030">__proto__</code>、实例对象之间的关系，这是很多初学者容易搞混的。根据这张图，我们可以得到以下的关系：</p>
<ol data-nodeid="1778">
<li data-nodeid="1779">
<p data-nodeid="1780">每个函数的原型对象（<code data-backticks="1" data-nodeid="2033">Person.prototype</code>）都拥有<code data-backticks="1" data-nodeid="2035">constructor</code>属性，指向该原型对象的构造函数（<code data-backticks="1" data-nodeid="2037">Person</code>）；</p>
</li>
<li data-nodeid="1781">
<p data-nodeid="1782">使用构造函数（<code data-backticks="1" data-nodeid="2040">new Person()</code>）可以创建对象，创建的对象称为实例对象（<code data-backticks="1" data-nodeid="2042">lily</code>）；</p>
</li>
<li data-nodeid="1783">
<p data-nodeid="1784">实例对象通过将<code data-backticks="1" data-nodeid="2045">__proto__</code>属性指向构造函数的原型对象（<code data-backticks="1" data-nodeid="2047">Person.prototype</code>），实现了该原型对象的继承。</p>
</li>
</ol>
<p data-nodeid="1785">那么现在，关于疑惑 2 中<code data-backticks="1" data-nodeid="2050">__proto__</code>和<code data-backticks="1" data-nodeid="2052">prototype</code>的关系，我们可以得到这样的答案：</p>
<ul data-nodeid="1786">
<li data-nodeid="1787">
<p data-nodeid="1788">每个对象都有<code data-backticks="1" data-nodeid="2055">__proto__</code>属性来标识自己所继承的原型对象，但只有函数才有<code data-backticks="1" data-nodeid="2057">prototype</code>属性；</p>
</li>
<li data-nodeid="1789">
<p data-nodeid="1790">对于函数来说，每个函数都有一个<code data-backticks="1" data-nodeid="2060">prototype</code>属性，该属性为该函数的原型对象；</p>
</li>
<li data-nodeid="1791">
<p data-nodeid="1792">通过将实例对象的<code data-backticks="1" data-nodeid="2063">__proto__</code>属性赋值为其构造函数的原型对象<code data-backticks="1" data-nodeid="2065">prototype</code>，JavaScript 可以使用构造函数创建对象的方式，来实现继承。</p>
</li>
</ul>
<p data-nodeid="1793">现在我们知道，一个对象可通过<code data-backticks="1" data-nodeid="2068">__proto__</code>访问原型对象上的属性和方法，而该原型同样也可通过<code data-backticks="1" data-nodeid="2070">__proto__</code>访问它的原型对象，这样我们就在实例和原型之间构造了一条原型链。这里我用红色的线将<code data-backticks="1" data-nodeid="2072">lily</code>实例的原型链标了出来。</p>
<p data-nodeid="1794"><img src="https://s0.lgstatic.com/i/image6/M01/39/CF/CioPOWB87jeAG0OeAADy6IPqiP8527.png" alt="图片3.png" data-nodeid="2076"></p>
<p data-nodeid="1795">下面一起来进行疑惑 4 “JavaScript 是怎么访问对象的方法和属性的？”的解答：在 JavaScript 中，是通过遍历原型链的方式，来访问对象的方法和属性。</p>
<h3 data-nodeid="1796">通过原型链访问对象的方法和属性</h3>
<p data-nodeid="1797">当 JavaScript 试图访问一个对象的属性时，会基于原型链进行查找。查找的过程是这样的：</p>
<ul data-nodeid="1798">
<li data-nodeid="1799">
<p data-nodeid="1800">首先会优先在该对象上搜寻。如果找不到，还会依次层层向上搜索该对象的原型对象、该对象的原型对象的原型对象等（套娃告警）；</p>
</li>
<li data-nodeid="1801">
<p data-nodeid="1802">JavaScript 中的所有对象都来自<code data-backticks="1" data-nodeid="2082">Object</code>，<code data-backticks="1" data-nodeid="2084">Object.prototype.__proto__ === null</code>。<code data-backticks="1" data-nodeid="2086">null</code>没有原型，并作为这个原型链中的最后一个环节；</p>
</li>
<li data-nodeid="1803">
<p data-nodeid="1804">JavaScript 会遍历访问对象的整个原型链，如果最终依然找不到，此时会认为该对象的属性值为<code data-backticks="1" data-nodeid="2089">undefined</code>。</p>
</li>
</ul>
<p data-nodeid="1805">我们可以通过一个具体的例子，来表示基于原型链的对象属性的访问过程，在该例子中我们构建了一条对象的原型链，并进行属性值的访问：</p>
<pre class="lang-java" data-nodeid="1806"><code data-language="java"><span class="hljs-comment">// 让我们假设我们有一个对象 o, 其有自己的属性 a 和 b：</span>
<span class="hljs-keyword">var</span> o = {a: <span class="hljs-number">1</span>, b: <span class="hljs-number">2</span>};
<span class="hljs-comment">// o 的原型 o.__proto__有属性 b 和 c：</span>
o.__proto__ = {b: <span class="hljs-number">3</span>, c: <span class="hljs-number">4</span>};
<span class="hljs-comment">// 最后, o.__proto__.__proto__ 是 null.</span>
<span class="hljs-comment">// 这就是原型链的末尾，即 null，</span>
<span class="hljs-comment">// 根据定义，null 没有__proto__.</span>
<span class="hljs-comment">// 综上，整个原型链如下:</span>
{a:<span class="hljs-number">1</span>, b:<span class="hljs-number">2</span>} ---&gt; {b:<span class="hljs-number">3</span>, c:<span class="hljs-number">4</span>} ---&gt; <span class="hljs-keyword">null</span>
<span class="hljs-comment">// 当我们在获取属性值的时候，就会触发原型链的查找：</span>
console.log(o.a); <span class="hljs-comment">// o.a =&gt; 1</span>
console.log(o.b); <span class="hljs-comment">// o.b =&gt; 2</span>
console.log(o.c); <span class="hljs-comment">// o.c =&gt; o.__proto__.c =&gt; 4</span>
console.log(o.d); <span class="hljs-comment">// o.c =&gt; o.__proto__.d =&gt; o.__proto__.__proto__ == null =&gt; undefined</span>
</code></pre>
<p data-nodeid="1807">可以看到，当我们对对象进行属性值的获取时，会触发该对象的原型链查找过程。</p>
<p data-nodeid="1808">既然 JavaScript 中会通过遍历原型链来访问对象的属性，那么我们可以通过原型链的方式进行继承。</p>
<p data-nodeid="1809">也就是说，可以通过原型链去访问原型对象上的属性和方法，我们不需要在创建对象的时候给该对象重新赋值/添加方法。比如，我们调用<code data-backticks="1" data-nodeid="2095">lily.toString()</code>时，JavaScript 引擎会进行以下操作：</p>
<ol data-nodeid="1810">
<li data-nodeid="1811">
<p data-nodeid="1812">先检查<code data-backticks="1" data-nodeid="2098">lily</code>对象是否具有可用的<code data-backticks="1" data-nodeid="2100">toString()</code>方法；</p>
</li>
<li data-nodeid="1813">
<p data-nodeid="1814">如果没有，则``检查<code data-backticks="1" data-nodeid="2106">lily</code>的原型对象（<code data-backticks="1" data-nodeid="2108">Person.prototype</code>）是否具有可用的<code data-backticks="1" data-nodeid="2110">toString()</code>方法；</p>
</li>
<li data-nodeid="1815">
<p data-nodeid="1816">如果也没有，则检查<code data-backticks="1" data-nodeid="2113">Person()</code>构造函数的<code data-backticks="1" data-nodeid="2115">prototype</code>属性所指向的对象的原型对象（即<code data-backticks="1" data-nodeid="2117">Object.prototype</code>）是否具有可用的<code data-backticks="1" data-nodeid="2119">toString()</code>方法，于是该方法被调用。</p>
</li>
</ol>
<p data-nodeid="1817">由于通过原型链进行属性的查找，需要层层遍历各个原型对象，此时可能会带来性能问题：</p>
<ul data-nodeid="1818">
<li data-nodeid="1819">
<p data-nodeid="1820">当试图访问不存在的属性时，会遍历整个原型链；</p>
</li>
<li data-nodeid="1821">
<p data-nodeid="1822">在原型链上查找属性比较耗时，对性能有副作用，这在性能要求苛刻的情况下很重要。</p>
</li>
</ul>
<p data-nodeid="1823">因此，我们在设计对象的时候，需要注意代码中原型链的长度。当原型链过长时，可以选择进行分解，来避免可能带来的性能问题。</p>
<p data-nodeid="1824">除了通过原型链的方式实现 JavaScript 继承，JavaScript 中实现继承的方式还包括经典继承(盗用构造函数)、组合继承、原型式继承、寄生式继承，等等。</p>
<ul data-nodeid="1825">
<li data-nodeid="1826">
<p data-nodeid="1827">原型链继承方式中引用类型的属性被所有实例共享，无法做到实例私有；</p>
</li>
<li data-nodeid="1828">
<p data-nodeid="1829">经典继承方式可以实现实例属性私有，但要求类型只能通过构造函数来定义；</p>
</li>
<li data-nodeid="1830">
<p data-nodeid="1831">组合继承融合原型链继承和构造函数的优点，它的实现如下：</p>
</li>
</ul>
<pre class="lang-java te-preview-highlight" data-nodeid="3873"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">Parent</span><span class="hljs-params">(name)</span> </span>{
  <span class="hljs-comment">// 私有属性，不共享</span>
  <span class="hljs-keyword">this</span>.name = name;
}

<span class="hljs-comment">// 需要复用、共享的方法定义在父类原型上</span>
Parent.prototype.speak = function() {
  console.log(<span class="hljs-string">"hello"</span>);
};

<span class="hljs-function">function <span class="hljs-title">Child</span><span class="hljs-params">(name)</span> </span>{
  Parent.call(<span class="hljs-keyword">this</span>, name);
}

<span class="hljs-comment">// 继承方法</span>
Child.prototype = <span class="hljs-keyword">new</span> Parent();
</code></pre>



<p data-nodeid="1833">组合继承模式通过将共享属性定义在父类原型上、将私有属性通过构造函数赋值的方式，实现了按需共享对象和方法，是 JavaScript 中最常用的继承模式。</p>
<p data-nodeid="1834">虽然在继承的实现方式上有很多种，但实际上都离不开原型对象和原型链的内容，因此掌握<code data-backticks="1" data-nodeid="2131">__proto__</code>和<code data-backticks="1" data-nodeid="2133">prototype</code>、对象的继承等这些知识，是我们实现各种继承方式的前提。</p>
<h3 data-nodeid="1835">小结</h3>
<p data-nodeid="1836">关于 JavaScript 的原型和继承，常常会在我们面试题中出现。随着 ES6/ES7 等新语法糖的出现，我们在日常开发中可能更倾向于使用<code data-backticks="1" data-nodeid="2137">class</code>/<code data-backticks="1" data-nodeid="2139">extends</code>等语法来编写代码，原型继承等概念逐渐变淡。</p>
<p data-nodeid="1837">但不管语法糖怎么先进，JavaScript 的设计在本质上依然没有变化，依然是基于原型来实现继承的。如果不了解这些内容，可能在我们遇到一些超出自己认知范围的内容时，很容易束手无策。</p>
<p data-nodeid="1838">现在，本文开始的四个疑惑我都在文中进行解答了，现在该轮到你了：</p>
<ol data-nodeid="1839">
<li data-nodeid="1840">
<p data-nodeid="1841">JavaScript 的函数和对象是怎样的关系？</p>
</li>
<li data-nodeid="1842">
<p data-nodeid="1843"><code data-backticks="1" data-nodeid="2144">__proto__</code>和<code data-backticks="1" data-nodeid="2146">prototype</code>都表示原型对象，它们有什么区别呢？</p>
</li>
<li data-nodeid="1844">
<p data-nodeid="1845">JavaScript 中对象的继承和原型链是什么关系？</p>
</li>
</ol>
<p data-nodeid="1846" class="">把你的想法写在留言区~</p>

---

### 精选评论

##### Better：
> 老师讲的好棒👍🏻1. 函数是一种特殊的对象，在对象内部属性拥有仅供 JavaScript引擎读取的 Call 属性的对象称为函数，使用 typeof 检测时会被识别为 function 。2. proto 可以称作指针指向 prototype ，后者实质上也是对象。3. 可以将 proto 比作链，prototype 比作节点，以 null 为顶点链接起来形成原型链，当访问标识符时，实例没有则会去原型链上查找，找到则返回结果，直到顶端 null 没找到则返回 undefined。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 你也好棒！

##### **2951：
> 1、Function instanceof Object === true;2、只有函数才有prototype 只有对象才有__proto__;3、一个对象的__proto__指向了另一个对象， 另一个对象的__proto__又指向了其他对象，举例let a = {name : "a"}let b = {age: 12}let c={}c.__proto__ = bb.__proto__ = a此时 c继承了a 和 b b继承了 a，同时他们的继承关系组成了一条原型链

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没毛病！

##### *聪：
> 对于构造函数，原型对象等概念不清晰的同学可以看看我的CSDN上的博客（看完一定懂）：《帮你彻底搞懂JS中的prototype、__proto__与constructor（图解）》，没错，原创：码飞_CC，就是我啦~~

##### **雄：
> 被删老师你好，我对于”函数的 `prototype` 属性指向它的原型对象“这句话有不同的看法；在此之前你说每个对象的 `__proto__` 指向它的原型对象，我是比较赞成的，所以对于函数，它的原型对象也应该是由 `__proto__` 指向的。那么函数的 `prototype` 要怎么理解的，它应该指向函数的实例对象的原型对象；对于一个函数，它的原型对象应该是 fn.__proto__ = Function.prototype , 也就是对于内置的构造函数 Function 的 prototype 指向它的实例对象 fn 的原型对象；以上是我结合老师讲解后，觉得有一丢丢矛盾的地方，进行了一点点自己的理解，不知道是否有偏差，希望老师能点评一下，万分感谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 文中应该没有说 prototype 属性指向它的原型对象？prototype 属性可以理解为就是函数的原型对象。 Function.prototype 在实例化之前就存在了，而 fn.__proto__ = Function.prototype 是在实例化过程中，将实例的 __proto__ 属性指向 Function.prototype 从而构成原型链。函数本身、以及函数的实例，这两者需要区分清楚~
因此，你说的“对于内置的构造函数 Function 的 prototype 指向它的实例对象 fn 的原型对象”，个人认为这样可能更加准确：“fn 这个实例，它的 __proto__ 指向它的构造函数的原型对象，即 Function.prototype”。

##### **怿：
> Person.__proto__ === lily.__proto__ ?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; lily.__proto__ === Person.prototype；
Person.prototype.__proto__ === Object.prototype；

##### *聪：
> 1. JavaScript中的函数也是一种对象。除了七种基本类型值，其他的所有都是对象，这就是JS中所谓的万物皆对象。2.__ptoto__属性是对象独有的，prototype属性是函数所独有的，因为函数也是一种对象，所以函数既有__proto__属性，也具有prototype属性，这点需要细细品味！3.对象的继承是依靠原型链来实现的，通过原型链，我们才可以使用其他对象上的属性或方法。

##### **远：
> 其实 __proto__ 属性是 chrome 自己搞出来的，没有被标准化，并且最新的 chrome 浏览器已经弃用这个属性了，改为 prototype 表示私有属性。标准中有专门用来访问对象原型的方法啊，就是 Object.getPrototypeOf()，标准提供了 Get/SetPrototypeOf 这两个方法用来操作对象的原型，应该避免使用 __proto__ 属性。继承一个对象的话，也是推荐 Object.create 方法，避免使用 __proto__ 属性。

##### *山：
> 当原型链过长时，可以选择进行分解，来避免可能带来的性能问题，请问怎么分解？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 避免使用过长的原型链就可以，比如不使用过深的继承关系

##### Kerita：
> 请问 Person.__proto__ 是什么东西？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Person.prototype.__proto__ === Object.prototype；
其实可以简单地去控制台打印看看的

##### **6082：
> 1.每一个构造函数都有一个prototype属性，指向函数的原型对象。并且当创建了一个构造函数后，其原型对象就会默认获得一个constructor属性，该属性解决了对象是由哪个构造函数创造出来的问题，即对象识别；2.每一个原型对象都有一个默认的constructor属性指向构造函数。除了constructor属性，还有__proto__指针；3.每一个对象都有一个__proto__属性，指向原型对象，也叫指针；4.构造函数的原型的原型是由Object生成的。即Foo.prototype.__proto__.constructor===Object 或者等价于Foo.prototype.__proto__===Object.prototype；5.原型链的终点是null,null不再有__proto__指针了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 妙呀

##### **玉：
> 1 都是对象，函数是一个不具体的对象，而对象是一个具体的对象，类似树与柳树的关系2 一个在函数上，一个在对象上3 继承依赖原型链，通过原型链来实现继承

##### **敏：
> 1.javascript中除了基础类型外，都是对象，函数是特殊的对象2.所有对象都有__proto__属性，指向它的构造函数的原型对象，每个函数都有个prototype属性，即原型对象3.原型链某种程度上就可以看做继承的表现

##### **茂：
> 你好，学了以后收货很多。现在还有两点疑问详情见下。1.你说只有函数才有prototype 那object. prototype 是咋回事？2.你打印的Lili的实例的那张图，我看到里面是有两个__proto__是怎么看出来__proto__等于prototype 的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 这里 Object.prototype 指 Object 的原型对象，并不是指 Object 的属性噢
2. 可以在控制台打印判断下噢，lily.__proto__ === Person.prototype

##### *忠：
> 求教，__proto__ 这个属性好像不是标准里面的吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 实际上，没有官方的方法用于直接访问一个对象的原型对象。在 JavaScript 语言标准中用 prototype 表示，然而大多数现代浏览器还是提供了 __proto__ 的属性来包含对象的原型

##### Diamonds：
> 课程多久一更啊 老师

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 一周两更~，每周一、三更新一节哦

##### Change：
> 催更！哈哈

