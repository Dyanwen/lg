<p data-nodeid="4705" class="">在自渲染模式中，Flutter 三棵树是一个比较关键的知识点。本课时将带你学习 Flutter 自渲染模式的三棵树，然后从三棵树的绘制过程中了解 Flutter 是如何做性能优化和如何进行 Flutter App 的性能提升。</p>
<h3 data-nodeid="4706">三棵树</h3>
<p data-nodeid="4707">在 Flutter 中存在三棵树，分别是 Widget 、Element 和 RenderObject。</p>
<ul data-nodeid="4708">
<li data-nodeid="4709">
<p data-nodeid="4710">Widget，是用来描述 UI 界面的，里面主要包含了一些基础的 UI 渲染的配置信息。</p>
</li>
<li data-nodeid="4711">
<p data-nodeid="4712">Element，类似于前端的虚拟 Dom，介于 Widget 和 RenderObject 之间。</p>
</li>
<li data-nodeid="4713">
<p data-nodeid="4714">RenderObject，则是实际上需要渲染的树，渲染引擎会根据 RenderObject 来进行界面渲染。</p>
</li>
</ul>
<p data-nodeid="4715">在 Flutter 中经过一系列处理后，将会生成一份这样的配置信息，如图 1 所示（你可以使用 debug 模式得到这份渲染树的结构信息）。</p>
<p data-nodeid="4716"><img src="https://s0.lgstatic.com/i/image/M00/44/E0/Ciqc1F8_i_CAfmbFAAJQmkbVhaU299.png" alt="Drawing 0.png" data-nodeid="4831"></p>
<div data-nodeid="4717"><p style="text-align:center">图 1 渲染树结构</p></div>
<p data-nodeid="4718">在图 1 中比较关键的是 3 个属性：</p>
<ul data-nodeid="4719">
<li data-nodeid="4720">
<p data-nodeid="4721">_widget 就是我们所说的 Widget 树；</p>
</li>
<li data-nodeid="4722">
<p data-nodeid="4723">_chilid 就是我们所说的 Element 树；</p>
</li>
<li data-nodeid="4724">
<p data-nodeid="4725">而 _renderObject 就是 RenderObject 树。</p>
</li>
</ul>
<p data-nodeid="4726">以上的渲染树结构对于我们所看到的 Widget 是一份非常简单的配置，如下：</p>
<pre class="lang-dart" data-nodeid="4727"><code data-language="dart"><span class="hljs-keyword">void</span> main() {
  runApp(MaterialApp(
    title: <span class="hljs-string">'Navigation Basics'</span>,
    home: FirstRoute(),
  ));
}
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">FirstRoute</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">StatelessWidget</span> </span>{
  <span class="hljs-meta">@override</span>
  Widget build(BuildContext context) {
    <span class="hljs-keyword">return</span> Center(
      child: Text(<span class="hljs-string">'flutter test'</span>),
    );
  }
}
</code></pre>
<p data-nodeid="4728">上面代码描述的是一个简单的页面组件，不过在这个简单页面组件背后是一个非常复杂的树型结构，具体看看渲染的 Element 树到底是个什么样子，如图 2 所示。</p>
<p data-nodeid="4729"><img src="https://s0.lgstatic.com/i/image/M00/44/E0/Ciqc1F8_i_uAHvbgAAFqAXyCe9s498.png" alt="Drawing 1.png" data-nodeid="4844"></p>
<div data-nodeid="4730"><p style="text-align:center">图 2 Element 树结构</p></div>
<p data-nodeid="4731">你有没有发现就一个非常简单的 Widget ，在 Flutter 中实际生成的 Element 树结构图是如此的复杂。你有没有发现在树的最底层才是我们使用的组件 FirstRoute-&gt;Center-&gt;Text-&gt;RichText（如图 2 中红色的部分）。了解完三棵树结构后，我们再来看下三棵树是如何进行转化的。</p>
<h3 data-nodeid="4732">三棵树对应关系</h3>
<p data-nodeid="4733">在 Flutter 中，Widget 和 Element 树是一一对应的，但是与 RenderObject 不是一一对应的。因为有些 Widget 是不需要渲染的，比如我们上面测试代码中的 FirstRoute 就是不需要渲染的 Widget。最终只有 RenderObjectWidget 相关的 Widget 才会转化为 RenderObject，也只有这种类型才需要进行渲染。可以看下表格 1 所展示的三棵树部分类型的对应关系。</p>
<p data-nodeid="4734"><img src="https://s0.lgstatic.com/i/image/M00/44/E0/Ciqc1F8_jAWASaH5AABxjuioTcw001.png" alt="Drawing 2.png" data-nodeid="4850"></p>
<div data-nodeid="4735"><p style="text-align:center">表格 1 Widget 、 Element 和 RenderObject 对应关系</p></div>
<p data-nodeid="4736">接下来我们看下三者是如何进行转化的。</p>
<h3 data-nodeid="4737">三棵树转化流程</h3>
<p data-nodeid="4738">Flutter 运行中的一部分核心逻辑就是在处理这三棵树的转化，所有的界面交互和事件处理，最终都反应在这三棵树上的操作结果。一般情况下，我们都是这样去运行 Flutter 项目的。</p>
<pre class="lang-dart" data-nodeid="4739"><code data-language="dart"><span class="hljs-keyword">void</span> main() {
  runApp(MaterialApp(
    title: <span class="hljs-string">'Navigation Basics'</span>,
    home: FirstRoute(),
  ));
}
</code></pre>
<p data-nodeid="4740">其中的 MaterialApp 就是我们所描述的一个 Widget ，Flutter 会经过 scheduleAttachRootWidget 、 attachRootWidget 、attachToRenderTree 调用到 RenderObjectToWidgetElement 的 mount 方法。在过程中会涉及相当多的源码函数，这里我们选择几个比较重要的函数介绍下。</p>
<h4 data-nodeid="4741">重要函数说明</h4>
<p data-nodeid="4742">在介绍函数之前我们先来看下整体的架构流程图，如图 3 所示。</p>
<p data-nodeid="4743"><img src="https://s0.lgstatic.com/i/image/M00/44/E0/Ciqc1F8_jBaAPZtCAADE5IavI9E126.png" alt="Drawing 3.png" data-nodeid="4859"></p>
<div data-nodeid="4744"><p style="text-align:center">图 3 Flutter 树转化图</p></div>
<p data-nodeid="4745">上述图比较复杂，你可以先简单了解下，等下我们会详细拆分来讲解。我们先来看下这几个关键函数的作用。</p>
<ul data-nodeid="4746">
<li data-nodeid="4747">
<p data-nodeid="4748"><strong data-nodeid="4865">scheduleAttachRootWidget</strong>，创建根 widget ，并且从根 widget 向子节点递归创建元素Element，对子节点为 RenderObjectWidget 的小部件创建 RenderObject 树节点，从而创建出 View 的渲染树，这里源代码中使用 Timer.run 事件任务的方式来运行，目的是避免影响到微任务的执行。</p>
</li>
</ul>
<pre class="lang-dart" data-nodeid="4749"><code data-language="dart"><span class="hljs-keyword">void</span> scheduleAttachRootWidget(Widget rootWidget) {
  Timer.run(() {
    attachRootWidget(rootWidget);
  });
}
</code></pre>
<ul data-nodeid="4750">
<li data-nodeid="4751">
<p data-nodeid="4752"><strong data-nodeid="4870">attachRootWidget</strong> 与 scheduleAttachRootWidget 作用一致，首先是创建根节点，然后调用 attachToRenderTree 循环创建子节点。</p>
</li>
</ul>
<pre class="lang-dart" data-nodeid="4753"><code data-language="dart"><span class="hljs-keyword">void</span> attachRootWidget(Widget rootWidget) {
  _readyToProduceFrames = <span class="hljs-keyword">true</span>;
  _renderViewElement = RenderObjectToWidgetAdapter&lt;RenderBox&gt;(
    container: renderView,
    debugShortDescription: <span class="hljs-string">'[root]'</span>,
    child: rootWidget,
  ).attachToRenderTree(buildOwner, renderViewElement <span class="hljs-keyword">as</span> RenderObjectToWidgetElement&lt;RenderBox&gt;);
}
</code></pre>
<ul data-nodeid="4754">
<li data-nodeid="4755">
<p data-nodeid="4756"><strong data-nodeid="4875">attachToRenderTree</strong>，该方法中有两个比较关键的调用，我只举例出核心代码部分，这里会先执行 buildScope ，但是在 buildScope 中会优先调用第二个参数（回调函数，也就是 element.mount ），而 mount 就会循环创建子节点，并在创建的过程中将需要更新的数据标记为 dirty。</p>
</li>
</ul>
<pre class="lang-dart" data-nodeid="4757"><code data-language="dart">owner.buildScope(element, () {
  element.mount(<span class="hljs-keyword">null</span>, <span class="hljs-keyword">null</span>);
});
</code></pre>
<ul data-nodeid="4758">
<li data-nodeid="4759">
<p data-nodeid="4760"><strong data-nodeid="4880">buildScope</strong>，如果首次渲染 dirty 是空的列表，因此首次渲染在该函数中是没有任何执行流程的，该函数的核心还是在第二次渲染或者 setState 后，有标记 dirty 的 Element 时才会起作用，该函数的目的也是循环 dirty 数组，如果 Element 有 child 则会递归判断子元素，并进行子元素的 build ，创建新的 Element 或者修改 Element 或者创建 RenderObject。</p>
</li>
<li data-nodeid="4761">
<p data-nodeid="4762"><strong data-nodeid="4885">updateChild</strong>，该方法非常重要，所有子节点的处理都是经过该函数，在该函数中 Flutter 会处理 Element 与 RenderObject 的转化逻辑，通过 Element 树的中间状态来减少对 RenderObject 树的影响，从而提升性能。具体这个函数的代码逻辑，我们拆解来分析。该函数的输入参数，包括三个参数：Element child、Widget newWidget、dynamic newSlot 。child 为当前节点的 Element 信息， newWidget 为 Widget 树的新节点，newSlot 为节点的新位置。在了解参数后接下来看下核心逻辑，首先判断是否有新的 Widget 节点。</p>
</li>
</ul>
<pre class="lang-dart" data-nodeid="4763"><code data-language="dart"><span class="hljs-keyword">if</span> (newWidget == <span class="hljs-keyword">null</span>) {
  <span class="hljs-keyword">if</span> (child != <span class="hljs-keyword">null</span>)
    deactivateChild(child);
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
}
</code></pre>
<p data-nodeid="4764">如果不存在，则将当前节点的 Element 直接销毁，如果 Widget 存在该节点，并且 Element 中也存在该节点，那么就首先判断两个节点是否一致，如代码第一行，如果一致只是位置不同，则更新位置即可。其他情况下判断是否可更新子节点，如果可以则更新，如果不可以则销毁原来的 Element 子节点，并重新创建一个。</p>
<pre class="lang-dart" data-nodeid="4765"><code data-language="dart"><span class="hljs-keyword">if</span> (hasSameSuperclass &amp;&amp; child.widget == newWidget) {
  <span class="hljs-keyword">if</span> (child.slot != newSlot)
    updateSlotForChild(child, newSlot);
  newChild = child;
} <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (hasSameSuperclass &amp;&amp; Widget.canUpdate(child.widget, newWidget)) {
  <span class="hljs-keyword">if</span> (child.slot != newSlot)
    updateSlotForChild(child, newSlot);
  child.update(newWidget);
  <span class="hljs-keyword">assert</span>(child.widget == newWidget);
  <span class="hljs-keyword">assert</span>(() {
    child.owner._debugElementWasRebuilt(child);
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
  }());
  newChild = child;
} <span class="hljs-keyword">else</span> {
  deactivateChild(child);
  <span class="hljs-keyword">assert</span>(child._parent == <span class="hljs-keyword">null</span>);
  newChild = inflateWidget(newWidget, newSlot);
}
</code></pre>
<p data-nodeid="4766">上面代码的第 8 行非常关键，在 child.update 函数逻辑里面，会根据当前节点的类型，调用不同的 update ，可参考图 3 中的 update 下的流程，每一个流程也都会递归调用子节点，并循环返回到 updateChild 中。有以下三个核心的函数会重新进入 updateChild 流程中，分别是 performRebuild、inflateWidget 和 markNeedsBuild，接下来我们看下这三个函数具体的作用。</p>
<ul data-nodeid="4767">
<li data-nodeid="4768">
<p data-nodeid="4769"><strong data-nodeid="4892">performRebuild</strong>是非常关键的一个代码，这部分就是我们在组件中写的 build 逻辑函数，StatelessWidget 和 StatefulWidget 的 build 函数都是在此执行，执行完成后将作为该节点的子节点，并进入 updateChild 递归函数中。</p>
</li>
<li data-nodeid="4770">
<p data-nodeid="4771"><strong data-nodeid="4897">inflateWidget</strong>创建一个新的节点，在创建完成后会根据当前 Element 类型，判断是 RenderObjectElement 或者 ComponentElement 。根据两者类型的不同，调用不同 mount 挂载到当前节点上，在两种类型的 mount 中又会循环子节点，调用 updateChild 重新进入子节点更新流程。这里还有一点，当为 RenderObjectElement 的时候会去创建 RenderObject 。</p>
</li>
<li data-nodeid="4772">
<p data-nodeid="4773"><strong data-nodeid="4902">markNeedsBuild</strong>，标记为 dirty ，并且调用 scheduleBuildFor 等待下一次 buildScope 操作。</p>
</li>
</ul>
<p data-nodeid="4774">以上就是比较关键的几个函数，其他函数你可以自己查看官网的文档。下面结合图 3 的流程图，我结合两个流程来讲解：首次 build 的流程和 setState 的流程。</p>
<h4 data-nodeid="4775">首次 build</h4>
<p data-nodeid="4776">当我们首次加载一个页面组件的时候，由于所有节点都是不存在的，因此这时候的流程大部分情况下都是创建新的节点，流程会如图 4 所示。</p>
<p data-nodeid="4777"><img src="https://s0.lgstatic.com/i/image/M00/44/E0/Ciqc1F8_jEiAbM-GAAFJbtV_XLo642.png" alt="Drawing 4.png" data-nodeid="4908"></p>
<div data-nodeid="4778"><p style="text-align:center">图 4 首次 build 流程</p></div>
<p data-nodeid="4779">runApp 到 RenderObjectToWidgetElement(mount) 逻辑都是一样的，在 _rebuild 中会调用 updateChild 更新节点，由于节点是不存在的，因此这时候就调用 inflateWidget 来创建 Element。</p>
<p data-nodeid="4780">当 <strong data-nodeid="4919">Element 为 Component</strong> 时，会调用 Component.mount ，在 Component.mount 中会创建 Element 并挂载到当前节点上，其次会调用 _firstBuild 进行子组件的 build ，build 完成后则将 build 好的组件作为子组件，进入 updateChild 的子组件更新。</p>
<p data-nodeid="4781">当 <strong data-nodeid="4925">Element 为 RenderObjectElement</strong> 时，则会调用 RenderObjectElement.mount，在 RenderObjectElement.mount 中会创建 RenderObjectElement 并且调用 createRenderObject 创建 RenderObject，并将该 RenderObject 和 RenderObjectElement 分别挂载到当前节点的 Element 树和 RenderObject 树，最后同样会调用 updateChild 来递归创建子节点。</p>
<p data-nodeid="4782">以上就是首次 build 的逻辑，单独来看还是非常清晰的，接下来我们看下 setState 的逻辑。</p>
<h4 data-nodeid="4783">setState</h4>
<p data-nodeid="4784">当我们调用 setState 后，我们实际上调用的是组件的 markNeedsBuild ，而这个函数上面已经介绍到是将组件设置为 dirty ，并且添加下一次 buildScope 的逻辑，等待下一次 rebuild 循环。如图 5 流程。buildScope 会调用 rebuild 然后进入 build 操作，从而进入 updateChild 的循环体系。</p>
<p data-nodeid="4785"><img src="https://s0.lgstatic.com/i/image/M00/44/E0/Ciqc1F8_jJGATAlDAAFF3eMI6D8595.png" alt="Drawing 6.png" data-nodeid="4931"></p>
<div data-nodeid="4786"><p style="text-align:center">图 5 setState 流程</p></div>
<p data-nodeid="4787">在图 5 中，我们就可以了解到，在 Flutter 中，如果父节点更新以后，也就是 setState 调用必定会引起子节点的递归循环判断 build 逻辑，虽然不一定会进行 RenderObject 树的创建（因为可能子节点没有变化，因此没有改变），但还是存在一定的性能影响。</p>
<p data-nodeid="4788">以上就是三棵树的转化过程，其中我省略了部分非核心流程函数，大家如果感兴趣，可以在<a href="https://github.com/flutter/flutter/tree/master/packages" data-nodeid="4936">Flutter 官网的 Github</a> 上进行学习。在掌握了整体的流程，我们接下来就要从这个转化过程中，提炼出可以提升我们 Flutter APP 性能的关键点。</p>
<h3 data-nodeid="4789">性能提升要点</h3>
<p data-nodeid="4790">在图 3 的整体流程中，我们要特别注意的就是 updateChild 这个函数，这也是 Flutter 从 Widget 到 Element 再到 RenderObject 性能提升的关键点。这个函数的作用在上面已经有介绍到了，关键点就是在 Widget 转化为 Element，然后 Element 转化 RenderObject 过程中做的一些细节的判断优化，这些细节处理包括以下这五点。</p>
<ul data-nodeid="4791">
<li data-nodeid="4792">
<p data-nodeid="4793">新节点被删除了，则直接删除 Element 节点。</p>
</li>
<li data-nodeid="4794">
<p data-nodeid="4795">节点存在，组件类型相同，并且组件相等时，则更新节点位置。</p>
</li>
<li data-nodeid="4796">
<p data-nodeid="4797">节点存在，组件类型相同，组件不相同，并且组件可进行更新时，则更新组件，由于当前组件更新了，因此需要更新当前组件的子节点，所以调用 update 来更新子节点列表，在此过程中也会对节点的 RenderObject 的子节点进行更新。</p>
</li>
<li data-nodeid="4798">
<p data-nodeid="4799">节点存在，组件类型相同，组件不相同，其次也无法进行组件更新时，则创建节点，同时在创建过程中判断是否为 RenderObject ，如果是则创建 RenderObject ，并循环判断子节点。</p>
</li>
<li data-nodeid="4800">
<p data-nodeid="4801">节点不存在，则同样走创建流程。</p>
</li>
</ul>
<p data-nodeid="4802">通过这种方式就可以减少 Widget 对 RenderObject 的影响，只有需要创建和更新的节点才会反映到 RenderObject 树中。从这个树节点的转化过程，我们可以提炼出以下四个关键点，从而提升我们 APP 的性能。</p>
<h4 data-nodeid="4803">const</h4>
<p data-nodeid="4804">上面提到了父组件更新，导致子组件都需要进行 rebuild 操作，一种办法就是减少有状态组件下的子组件，还有一种办法就是尽量多用 const 组件，这样即使父组件更新了，子组件也不会重新 rebuild 操作。这里就是在上面的判断逻辑，节点存在，组件类型相同，并且组件相等时的处理逻辑。</p>
<p data-nodeid="4805">这点在我们项目的源代码中，也有一些实践优化的点，特别是一些长期不修改的组件，例如通用报错组件和通用 loading 组件等，当然只能针对不带变量的组件返回，例如下面这部分代码。</p>
<pre class="lang-dart" data-nodeid="4806"><code data-language="dart"><span class="hljs-keyword">if</span> (error) {
    <span class="hljs-keyword">return</span> CommonError(action: <span class="hljs-keyword">this</span>.setFirstPage);
  }
  <span class="hljs-keyword">if</span> (contentList == <span class="hljs-keyword">null</span>) {
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">const</span> Loading();
  }
}
</code></pre>
<p data-nodeid="4807">第 2 行代码中有一个变量 action ，因此是不能设置为 const 的，下面的 Loading 由于没有携带变量，因此是可以设置为 const 的。其他代码可以同样进行修改，对性能提升还是有一定的帮助的，特别是在组件设计不合理的过程中。</p>
<h4 data-nodeid="4808">canUpdate</h4>
<p data-nodeid="4809">在 updateChild 上面流程中，有一个执行函数 canUpdate ，这个也是一个性能提升的关键点，特别是在需要对多个元素进行调整时，可以看下具体的逻辑实现。</p>
<pre class="lang-dart" data-nodeid="4810"><code data-language="dart"><span class="hljs-keyword">static</span> <span class="hljs-built_in">bool</span> canUpdate(Widget oldWidget, Widget newWidget) {
  <span class="hljs-keyword">return</span> oldWidget.runtimeType == newWidget.runtimeType
      &amp;&amp; oldWidget.key == newWidget.key;
}
</code></pre>
<p data-nodeid="4811">主要是判断运行时的类是否相同，同时判断 key 是否一样，如果都一样，则可直接更新组件 Element 位置，提升性能，因此在组件设计时，尽量减少组件的 key 的变化，可以默认设置为空。</p>
<p data-nodeid="4812">其次在如果需要频繁地对组件进行排序、删除或者新增处理时，最好要将组件增加上 key ，以提升性能。这里要非常注意，由于 StatefulWidget 的 state 是保存在 Element 中，因此如果希望区分两个相同类名（ runtimeType ）的 Widget 时，必须携带不同的 key ，不然无法区分新旧 Widget 的变化，特别是在一个列表数据，每个列表都是一个有状态类，如果需要切换列表中项目列表时，则必须设置 key，不然会导致顺序切换失效。了解更多关于这点的，可以参考这篇<a href="https://medium.com/flutter/keys-what-are-they-good-for-13cb51742e7d" data-nodeid="4956">英文的文章</a>。</p>
<h4 data-nodeid="4813">inflateWidget</h4>
<p data-nodeid="4814">在 updateChild 中的 inflateWidget 执行函数也是一个比较关键的性能提升点，这个函数在创建之前会检查 key 是否为 GlobalKey ，如果是则表明 Element 存在，那么这时候直接启用即可，如果不存在则需要重新创建，这就类似与组件缓存，只能说减少组件的 build 成本，看下如下这部分代码。</p>
<pre class="lang-dart" data-nodeid="4815"><code data-language="dart"><span class="hljs-keyword">final</span> Key key = newWidget.key;
<span class="hljs-keyword">if</span> (key <span class="hljs-keyword">is</span> GlobalKey) {
  <span class="hljs-keyword">final</span> <span class="hljs-built_in">Element</span> newChild = _retakeInactiveElement(key, newWidget);
  <span class="hljs-keyword">if</span> (newChild != <span class="hljs-keyword">null</span>) {
    <span class="hljs-keyword">assert</span>(newChild._parent == <span class="hljs-keyword">null</span>);
    <span class="hljs-keyword">assert</span>(() {
      _debugCheckForCycles(newChild);
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
    }());
    newChild._activateWithParent(<span class="hljs-keyword">this</span>, newSlot);
    <span class="hljs-keyword">final</span> <span class="hljs-built_in">Element</span> updatedChild = updateChild(newChild, newWidget, newSlot);
    <span class="hljs-keyword">assert</span>(newChild == updatedChild);
    <span class="hljs-keyword">return</span> updatedChild;
  }
}
<span class="hljs-keyword">final</span> <span class="hljs-built_in">Element</span> newChild = newWidget.createElement();
</code></pre>
<p data-nodeid="4816">但是这部分也是非常损耗内存的，因为它会将组件缓存到内存中，导致垃圾内容无法进行回收，因此在使用 GlobalKey 要非常注意，尽量应用在复用性高且 build 业务复杂的组件上。</p>
<h4 data-nodeid="4817">setState</h4>
<p data-nodeid="4818">在图 3 中的 setState 被触发后，当前组件会进行 rebuild 操作，由于当前组件的 build ，会引起当前组件下的所有子组件发生 rebuild 行为，因此在设计时，<strong data-nodeid="4967">尽量减少有状态组件下的无状态组件</strong>，从而减少没必要的 build 逻辑。这也是我们之前提到的一些组件设计要点，虽然说 Flutter 构建 Widget 和 Element 是比较快的，但是为了性能考虑，还是尽量减少这部分没必要的损耗。其次也注意减少 build 中的业务逻辑，因为 Flutter 中的 build 是会经常被触发，特别是有状态组件。</p>
<h3 data-nodeid="4819">总结</h3>
<p data-nodeid="4820">本课时着重介绍了 Flutter 自渲染中的三棵树知识，从 Flutter 的三棵树概念到三棵树对应关系，其中着重介绍了三棵树的转化流程，并在流程中总结出性能优化需要着重注意的点。</p>
<p data-nodeid="10069">学完本课时后，你需要掌握 Flutter 的三棵树概念，并非常清晰的了解三棵树的转化过程，通过对转化过程中性能优化知识的学习，从而在编码过程中养成一个非常好的编码习惯。</p>
<p data-nodeid="34281" class=""><a href="https://github.com/love-flutter/flutter-column" data-nodeid="34284">点击链接，查看本课时源码</a></p>

---

### 精选评论

##### **龙：
> 尽量减少有状态组件下的无状态组件，从而减少没必要的 build 逻辑。老师这个没太理解，这意思是说stateful里面的字widget也要使用statefulWidget？？我都是能用stateless就不用stateful的😁

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 首先你的做法是ok的，能用stateless 就不用stateful。我的那句话的意思是说：当你需要设置一个有状态组件时，尽量减少这个有状态组件下的子组件，比如我们专栏中的例子，为什么我们需要把有状态组件设置在最小的组件部分 article_like_bar 而不是将整个article_card 设为有状态组件的原因。因为有状态组件的任何更新操作，都会导致所有子组件的 rebuild 逻辑。所以只将有状态的部分，设置为一个最小单元的有状态组件。

