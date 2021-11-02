<p data-nodeid="3145" class="">前端页面的布局和样式编写是传统技能，但页面样式的实现大多数情况下都无法速成，需要通过不断练习、反复地调试才能熟练掌握，因此有一些同学常常会感到疑惑，比如：</p>
<ol data-nodeid="3146">
<li data-nodeid="3147">
<p data-nodeid="3148">一个元素总宽高为<code data-backticks="1" data-nodeid="3373">50px</code>，要怎么在调整边框大小的时候，不需要重新计算和设置<code data-backticks="1" data-nodeid="3375">width/height</code>呢？</p>
</li>
<li data-nodeid="3149">
<p data-nodeid="3150">为什么给一些元素设置宽高，但是却不生效？</p>
</li>
<li data-nodeid="3151">
<p data-nodeid="3152">如何将一个元素固定在页面的某个位置，具体怎么做？</p>
</li>
<li data-nodeid="3153">
<p data-nodeid="3154">为什么将某个元素<code data-backticks="1" data-nodeid="3380">z-index</code>设置为<code data-backticks="1" data-nodeid="3382">9999999</code>，但是它依然被其他元素遮挡住了呢？</p>
</li>
<li data-nodeid="3155">
<p data-nodeid="3156">为什么将某个元素里面的元素设置为<code data-backticks="1" data-nodeid="3385">float</code>之后，这个元素的高度就歪了呢？</p>
</li>
<li data-nodeid="3157">
<p data-nodeid="3158">让一个元素进行垂直和水平居中，有多少种实现方式？</p>
</li>
</ol>
<p data-nodeid="3159">这些问题产生的根本，是对页面布局规则和常见页面布局方式没掌握透彻。今天我就帮你重新梳理下页面布局的基本规则和布局方式，让以上问题迎刃而解。</p>
<h3 data-nodeid="3160">页面布局的基本规则</h3>
<p data-nodeid="3161">我们在调试页面样式的时候，如果你不了解页面布局规则，会经常遇到“这里为什么歪了”“这里为什么又好了”这样的困惑。其实页面的布局不只是“碰运气”似的调整样式，浏览器的页面布局会有一些规则，包括：</p>
<ul data-nodeid="3162">
<li data-nodeid="3163">
<p data-nodeid="3164">盒模型计算；</p>
</li>
<li data-nodeid="3165">
<p data-nodeid="3166">内联元素与块状元素布局规则；</p>
</li>
<li data-nodeid="3167">
<p data-nodeid="3168">文档流布局；</p>
</li>
<li data-nodeid="3169">
<p data-nodeid="3170">元素堆叠。</p>
</li>
</ul>
<p data-nodeid="3171">下面我们可以结合问题逐一来看。</p>
<h4 data-nodeid="3172">盒模型计算</h4>
<p data-nodeid="3173">问题 1：一个元素总宽高为<code data-backticks="1" data-nodeid="3398">30px</code>，要怎么在调整边框大小的时候，不需要重新计算和设置<code data-backticks="1" data-nodeid="3400">width/height</code>呢？</p>
<p data-nodeid="3174">这个问题涉及浏览器布局中的盒模型计算。什么是盒模型？浏览器对文档进行布局的时候，会将每个元素都表示为这样一个盒子。</p>
<p data-nodeid="3175"><img src="https://s0.lgstatic.com/i/image6/M00/33/F7/Cgp9HWBwBrWAfuU6AANyH4P_TXw391.png" alt="Drawing 1.png" data-nodeid="3405"></p>
<p data-nodeid="3176">这就是 CSS 基础盒模型，也就是我们常说的盒模型。盒模型主要用来描述元素所占空间的内容，它由四个部分组成：</p>
<ul data-nodeid="3177">
<li data-nodeid="3178">
<p data-nodeid="3179">外边框边界<code data-backticks="1" data-nodeid="3408">margin</code>（橙色部分）</p>
</li>
<li data-nodeid="3180">
<p data-nodeid="3181">边框边界<code data-backticks="1" data-nodeid="3411">border</code>（黄色部分）</p>
</li>
<li data-nodeid="3182">
<p data-nodeid="3183">内边距边界<code data-backticks="1" data-nodeid="3414">padding</code>（绿色部分）</p>
</li>
<li data-nodeid="3184">
<p data-nodeid="3185">内容边界<code data-backticks="1" data-nodeid="3417">content</code>（蓝色部分）</p>
</li>
</ul>
<p data-nodeid="3186">盒模型是根据元素的样式来进行计算的，我们可以通过调整元素的样式来改变盒模型。上图中的盒模型来自下面这个<code data-backticks="1" data-nodeid="3420">&lt;div&gt;</code>元素，我们给这个元素设置了<code data-backticks="1" data-nodeid="3422">margin</code>、<code data-backticks="1" data-nodeid="3424">padding</code>和<code data-backticks="1" data-nodeid="3426">border</code>：</p>
<pre class="lang-plain" data-nodeid="3187"><code data-language="plain">&lt;style&gt;
  .box-model-sample {
    margin: 10px;
    padding: 10px;
    border: solid 2px #000;
  }
&lt;/style&gt;
&lt;div class="box-model-sample"&gt;这是一个div&lt;/div&gt;
</code></pre>
<p data-nodeid="3188">在上述代码中，我们通过使用 CSS 样式来控制盒模型的大小和属性。盒模型还常用来控制元素的尺寸、属性（颜色、背景、边框等）和位置，当我们在调试样式的时，比较容易遇到以下这些场景。</p>
<p data-nodeid="3189"><strong data-nodeid="3439">1</strong>. 盒模型会发生<code data-backticks="1" data-nodeid="3433">margin</code>外边距叠加，叠加后的值会以最大边距为准。比如，我们给两个相邻的<code data-backticks="1" data-nodeid="3435">&lt;div&gt;</code>元素分别设置了不同的<code data-backticks="1" data-nodeid="3437">margin</code>外边距：</p>
<pre class="lang-plain" data-nodeid="3190"><code data-language="plain">&lt;style&gt;
  .box-model-sample {
    margin: 10px;
    padding: 10px;
    border: solid 2px #000;
  }
  .large-margin {
    margin: 20px;
  }
&lt;/style&gt;
&lt;div class="box-model-sample"&gt;这是一个div&lt;/div&gt;
&lt;div class="box-model-sample"&gt;这是另一个div&lt;/div&gt;
&lt;div class="box-model-sample large-margin"&gt;这是一个margin大一点的div&lt;/div&gt;
</code></pre>
<p data-nodeid="3191">这段代码在浏览器中运行时，我们可以看到，两个<code data-backticks="1" data-nodeid="3441">&lt;div&gt;</code>元素之间发生了<code data-backticks="1" data-nodeid="3443">margin</code>外边距叠加，它们被合并成单个边距。</p>
<p data-nodeid="3192"><img src="https://s0.lgstatic.com/i/image6/M00/34/00/CioPOWBwBsGATe5fAACdV1B5j8s079.png" alt="Drawing 3.png" data-nodeid="3447"></p>
<p data-nodeid="3193">如果两个元素的外边距不一样，叠加的值大小是各个边距中的最大值，比如上面第二个和第三个矩形之间的外边距值，使用的是第三个边框的外边距值 20 px。</p>
<p data-nodeid="3194"><img src="https://s0.lgstatic.com/i/image6/M00/34/00/CioPOWBwBseATZypAACnWIPJ5fU407.png" alt="Drawing 5.png" data-nodeid="3451"><br>
需要注意的是，并不是所有情况下都会发生外边距叠加，比如行内框、浮动框或绝对定位框之间的外边距不会叠加。</p>
<p data-nodeid="3195"><strong data-nodeid="3462">2</strong>. 盒模型计算效果有多种，比如元素宽高是否包括了边框。我们可以通过<code data-backticks="1" data-nodeid="3458">box-sizing</code>属性进行设置盒模型的计算方式，正常的盒模型默认值是<code data-backticks="1" data-nodeid="3460">content-box</code>。</p>
<p data-nodeid="3196">使用<code data-backticks="1" data-nodeid="3464">box-sizing</code>属性可以解决问题 1（调整元素的边框时，不影响元素的宽高），我们可以将元素的<code data-backticks="1" data-nodeid="3466">box-sizing</code>属性设置为<code data-backticks="1" data-nodeid="3468">border-box</code>：</p>
<pre class="lang-js" data-nodeid="3197"><code data-language="js">&lt;style&gt;
  .box-model-sample {
    height: 50px;
    margin: 10px;
    padding: 5px;
    border: solid 2px #000;
  }
  .border-box {
    box-sizing: border-box;
  }
&lt;/style&gt;
&lt;div class="box-model-sample"&gt;这是一个div(content-box)&lt;/div&gt;
&lt;div class="box-model-sample border-box"&gt;这是另一个div(border-box)&lt;/div&gt;
</code></pre>
<p data-nodeid="3198">对于默认<code data-backticks="1" data-nodeid="3471">content-box</code>的元素来说，元素所占的总宽高为设置的元素宽高(<code data-backticks="1" data-nodeid="3473">width</code>/<code data-backticks="1" data-nodeid="3475">height</code>)等于：<code data-backticks="1" data-nodeid="3477">content + padding + border</code>，因此这里该元素总高度为<code data-backticks="1" data-nodeid="3479">50 + 5 * 2 + 2 * 2 = 64px</code>。</p>
<p data-nodeid="3199"><img src="https://s0.lgstatic.com/i/image6/M00/34/00/CioPOWBwBtmAHIF5AAC8NdjpmFw307.png" alt="Drawing 7.png" data-nodeid="3483"></p>
<p data-nodeid="3200">当我们设置为<code data-backticks="1" data-nodeid="3485">border-box</code>之后，元素所占的总宽高为设置的元素宽高(<code data-backticks="1" data-nodeid="3487">width</code>/<code data-backticks="1" data-nodeid="3489">height</code>)，因此，此时高度为<code data-backticks="1" data-nodeid="3491">50px</code>：</p>
<p data-nodeid="3201"><img src="https://s0.lgstatic.com/i/image6/M00/34/00/CioPOWBwBuCAPnYtAADeCHGecrY299.png" alt="Drawing 9.png" data-nodeid="3495"></p>
<p data-nodeid="3202">也就是说，如果我们在调整元素边框的时候，不影响元素的宽高，可以给元素的<code data-backticks="1" data-nodeid="3497">box-sizing</code>属性设置为<code data-backticks="1" data-nodeid="3499">border-box</code>，这便是问题 1 的答案。通过这种方式，我们可以精确地控制元素的空间占位，同时还能灵活地调整元素边框和内边距。</p>
<p data-nodeid="3203">虽然我们可以通过盒模型设置元素的占位情况，但是有些时候我们给元素设置宽高却不生效（见问题 2），这是因为元素本身的性质也做了区分，我们来看一下。</p>
<h4 data-nodeid="3204">内联元素与块状元素</h4>
<p data-nodeid="3205">在浏览器中，元素可分为内联元素和块状元素。比如，<code data-backticks="1" data-nodeid="3504">&lt;a&gt;</code>元素为内联元素，<code data-backticks="1" data-nodeid="3506">&lt;div&gt;</code>元素为块状元素，我们分别给它们设置宽高：</p>
<pre class="lang-java" data-nodeid="3206"><code data-language="java">&lt;style&gt;
  a,
  div {
    width: 100px;
    height: 20px;
  }
&lt;/style&gt;
&lt;a&gt;a-123&lt;/a&gt;&lt;a&gt;a-456&lt;/a&gt;&lt;a&gt;a-789&lt;/a&gt;
&lt;div&gt;div-123&lt;/div&gt;
&lt;div&gt;div-456&lt;/div&gt;
&lt;div&gt;div-789&lt;/div&gt;
</code></pre>
<p data-nodeid="3207">在浏览器中的效果如下图所示：</p>
<p data-nodeid="3208"><img src="https://s0.lgstatic.com/i/image6/M01/33/F8/Cgp9HWBwB1OAfXNsAAFw2Bp-aVw496.png" alt="Drawing 10.png" data-nodeid="3511"></p>
<p data-nodeid="3209">可以看到，<code data-backticks="1" data-nodeid="3513">&lt;a&gt;</code>元素和<code data-backticks="1" data-nodeid="3515">&lt;div&gt;</code>元素最主要的区别在于：</p>
<ul data-nodeid="3210">
<li data-nodeid="3211">
<p data-nodeid="3212"><code data-backticks="1" data-nodeid="3517">&lt;a&gt;</code>元素（内联元素）可以和其他内联元素位于同一行，且宽高设置无效；</p>
</li>
<li data-nodeid="3213">
<p data-nodeid="3214"><code data-backticks="1" data-nodeid="3519">&lt;div&gt;</code>元素（块状元素）不可和其他元素位于同一行，且宽高设置有效。</p>
</li>
</ul>
<p data-nodeid="3215">所以问题 2 的答案是，当我们给某个元素设置宽高不生效，是因为该元素为内联元素。那么有没有办法解决这个问题呢？</p>
<p data-nodeid="3216">我们可以通过设置<code data-backticks="1" data-nodeid="3523">display</code>的值来对元素进行调整。</p>
<ul data-nodeid="3217">
<li data-nodeid="3218">
<p data-nodeid="3219">设置为<code data-backticks="1" data-nodeid="3526">block</code>块状元素，此时可以设置宽度<code data-backticks="1" data-nodeid="3528">width</code>和高度<code data-backticks="1" data-nodeid="3530">height</code>。</p>
</li>
<li data-nodeid="3220">
<p data-nodeid="3221">设置为<code data-backticks="1" data-nodeid="3533">inline</code>内联元素，此时宽度高度不起作用。</p>
</li>
<li data-nodeid="3222">
<p data-nodeid="3223">设置为<code data-backticks="1" data-nodeid="3536">inline-block</code>，可以理解为块状元素和内联元素的结合，布局规则包括：</p>
<ul data-nodeid="3224">
<li data-nodeid="3225">
<p data-nodeid="3226">位于块状元素或者其他内联元素内；</p>
</li>
<li data-nodeid="3227">
<p data-nodeid="3228">可容纳其他块状元素或内联元素；</p>
</li>
<li data-nodeid="3229">
<p data-nodeid="3230">宽度高度起作用。</p>
</li>
</ul>
</li>
</ul>
<p data-nodeid="3231">除了内联元素和块状元素，我们还可以将元素设置为<code data-backticks="1" data-nodeid="3542">inline-block</code>，<code data-backticks="1" data-nodeid="3544">inline-block</code>可以很方便解决一些问题：使元素居中、给<code data-backticks="1" data-nodeid="3546">inline</code>元素（<code data-backticks="1" data-nodeid="3548">&lt;a&gt;</code>/<code data-backticks="1" data-nodeid="3550">&lt;span&gt;</code>）设置宽高、将多个块状元素放在一行等。</p>
<h4 data-nodeid="3232">文档流和元素定位</h4>
<p data-nodeid="3233">接下来，我们来看问题 3：将一个元素固定在页面的某个位置，可以怎么做？这个问题涉及文档流的布局和元素定位的样式设置。</p>
<p data-nodeid="3234">什么是文档流呢？正常的文档流在 HTML 里面为从上到下，从左到右的排版布局。</p>
<p data-nodeid="3235">文档流布局方式可以使用<code data-backticks="1" data-nodeid="3556">position</code>样式进行调整，包括：<code data-backticks="1" data-nodeid="3558">static</code>（默认值）、<code data-backticks="1" data-nodeid="3560">inherit</code>（继承父元素）、<code data-backticks="1" data-nodeid="3562">relative</code>（相对定位）、<code data-backticks="1" data-nodeid="3564">absolute</code>（相对非<code data-backticks="1" data-nodeid="3566">static</code>父元素绝对定位）、<code data-backticks="1" data-nodeid="3568">fixed</code>（相对浏览器窗口进行绝对定位）。</p>
<p data-nodeid="3236">我们来分别看下这些<code data-backticks="1" data-nodeid="3571">position</code>样式设置效果。</p>
<p data-nodeid="3237"><strong data-nodeid="3597">1</strong>. 元素<code data-backticks="1" data-nodeid="3577">position</code>样式属性值为<code data-backticks="1" data-nodeid="3579">static</code>(默认值)时，元素会忽略<code data-backticks="1" data-nodeid="3581">top</code>/<code data-backticks="1" data-nodeid="3583">bottom</code>/<code data-backticks="1" data-nodeid="3585">left</code>/<code data-backticks="1" data-nodeid="3587">right</code>或者<code data-backticks="1" data-nodeid="3589">z-index</code>声明，比如我们给部分元素设置<code data-backticks="1" data-nodeid="3591">position: static</code>的样式以及<code data-backticks="1" data-nodeid="3593">left</code>和<code data-backticks="1" data-nodeid="3595">top</code>定位 ：</p>
<pre class="lang-css" data-nodeid="3238"><code data-language="css"><span class="hljs-selector-tag">a</span>, <span class="hljs-selector-tag">p</span>, <span class="hljs-selector-tag">div</span> {
  <span class="hljs-attribute">border</span>: solid <span class="hljs-number">1px</span> red;
}
<span class="hljs-selector-class">.static</span> {
  <span class="hljs-attribute">position</span>: static;
  <span class="hljs-attribute">left</span>: <span class="hljs-number">100px</span>;
  <span class="hljs-attribute">top</span>: <span class="hljs-number">100px</span>;
}
</code></pre>
<p data-nodeid="3239">在<a href="https://about-position-1255459943.file.myqcloud.com/position-static.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="3601">浏览器</a>中，我们可以看到给<code data-backticks="1" data-nodeid="3603">position: static</code>的元素添加定位<code data-backticks="1" data-nodeid="3605">left: 100px; top: 100px;</code>是无效的。</p>
<p data-nodeid="3240"><img src="https://s0.lgstatic.com/i/image6/M01/33/F8/Cgp9HWBwB2yAK13GAAEoSLhXiR0106.png" alt="Drawing 12.png" data-nodeid="3609"></p>
<div data-nodeid="3241"><p style="text-align:center">（static 元素的定位设置无效果）</p></div>
<p data-nodeid="3242"><strong data-nodeid="3624">2</strong>. 元素<code data-backticks="1" data-nodeid="3614">position</code>样式属性值为<code data-backticks="1" data-nodeid="3616">relative</code>时，元素会保持原有文档流，但相对本身的原始位置发生位移，且会占用空间，比如我们给部分元素设置<code data-backticks="1" data-nodeid="3618">position: relative</code>样式以及<code data-backticks="1" data-nodeid="3620">left</code>和<code data-backticks="1" data-nodeid="3622">top</code>定位：</p>
<pre class="lang-css" data-nodeid="3243"><code data-language="css"><span class="hljs-selector-tag">a</span>, <span class="hljs-selector-tag">p</span>, <span class="hljs-selector-tag">div</span> {
  <span class="hljs-attribute">border</span>: solid <span class="hljs-number">1px</span> red;
}
<span class="hljs-selector-class">.relative</span> {
  <span class="hljs-attribute">position</span>: relative;
  <span class="hljs-attribute">left</span>: <span class="hljs-number">100px</span>;
  <span class="hljs-attribute">top</span>: <span class="hljs-number">100px</span>;
}
</code></pre>
<p data-nodeid="3244">在<a href="https://about-position-1255459943.file.myqcloud.com/position-relative.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="3628">浏览器</a>中，我们可以看到<code data-backticks="1" data-nodeid="3630">position: relative</code>的元素相对于其正常位置进行定位，元素占有原本位置（文档流中占有的位置与其原本位置相同），因此下一个元素会排到该元素后方。</p>
<p data-nodeid="3245"><img src="https://s0.lgstatic.com/i/image6/M01/33/F8/Cgp9HWBwB3mAFFqSAADs-8Jl61g905.png" alt="Drawing 14.png" data-nodeid="3634"></p>
<div data-nodeid="3246"><p style="text-align:center">(relative 定位的元素，定位设置可生效)</p></div>
<p data-nodeid="3247">这里有个需要注意的地方：虽然<code data-backticks="1" data-nodeid="3636">relative</code>元素占位与<code data-backticks="1" data-nodeid="3638">static</code>相同，但会溢出父元素，撑开整个页面。如下图所示，我们能看到<a href="https://about-position-1255459943.file.myqcloud.com/position-relative-occupation.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="3642">浏览器中</a><code data-backticks="1" data-nodeid="3643">relative</code>元素撑开父元素看到页面底部有滚动条。</p>
<p data-nodeid="3248"><img src="https://s0.lgstatic.com/i/image6/M01/33/F8/Cgp9HWBwB5-ANcUmAAC64A1Pkhk727.png" alt="Drawing 16.png" data-nodeid="3647"></p>
<div data-nodeid="3249"><p style="text-align:center">(relative 定位的元素，可撑开父元素)</p></div>
<p data-nodeid="3250">此时给父元素设置<code data-backticks="1" data-nodeid="3649">overflow: hidden;</code>则可以隐藏溢出部分。</p>
<p data-nodeid="3251"><img src="https://s0.lgstatic.com/i/image6/M00/33/F8/Cgp9HWBwB6iARwZZAABrqdhMZGc561.png" alt="Drawing 18.png" data-nodeid="3653"></p>
<p data-nodeid="3252">（通过设置<code data-backticks="1" data-nodeid="3655">overflow: hidden</code>可隐藏溢出部分元素）</p>
<p data-nodeid="3253"><strong data-nodeid="3679">3</strong>. 元素<code data-backticks="1" data-nodeid="3661">position</code>样式属性值为<code data-backticks="1" data-nodeid="3663">absolute</code>、且设置了定位（<code data-backticks="1" data-nodeid="3665">top</code>/<code data-backticks="1" data-nodeid="3667">bottom</code>/<code data-backticks="1" data-nodeid="3669">left</code>/<code data-backticks="1" data-nodeid="3671">right</code>）时，元素会脱离文档流，相对于其包含块来定位，且不占位，比如我们给<code data-backticks="1" data-nodeid="3673">position: absolute</code>的元素设置<code data-backticks="1" data-nodeid="3675">left</code>和<code data-backticks="1" data-nodeid="3677">top</code>定位 ：</p>
<pre class="lang-css" data-nodeid="3254"><code data-language="css"><span class="hljs-selector-class">.parent</span> {
  <span class="hljs-attribute">border</span>: solid <span class="hljs-number">1px</span> blue;
  <span class="hljs-attribute">width</span>: <span class="hljs-number">300px</span>;
}
<span class="hljs-selector-class">.parent</span> &gt; <span class="hljs-selector-tag">div</span> {
  <span class="hljs-attribute">border</span>: solid <span class="hljs-number">1px</span> red;
  <span class="hljs-attribute">height</span>: <span class="hljs-number">100px</span>;
  <span class="hljs-attribute">width</span>: <span class="hljs-number">300px</span>;
}
<span class="hljs-selector-class">.absolute</span> {
  <span class="hljs-attribute">position</span>: absolute;
  <span class="hljs-attribute">left</span>: <span class="hljs-number">100px</span>;
  <span class="hljs-attribute">height</span>: <span class="hljs-number">100px</span>;
}
</code></pre>
<p data-nodeid="3255">在<a href="https://about-position-1255459943.file.myqcloud.com/position-absolute.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="3683">浏览器</a>中，我们可以看到<code data-backticks="1" data-nodeid="3685">position: absolute</code>的元素不占位，因此下一个符合普通流的元素会略过<code data-backticks="1" data-nodeid="3687">absolute</code>元素排到其上一个元素的后方。</p>
<p data-nodeid="3256"><img src="https://s0.lgstatic.com/i/image6/M01/33/F8/Cgp9HWBwB7KARvTaAABaLe2qI4k309.png" alt="Drawing 20.png" data-nodeid="3691"></p>
<div data-nodeid="3257"><p style="text-align:center">（absolute 元素不占位）</p></div>
<p data-nodeid="3258"><strong data-nodeid="3708">4</strong>. 元素<code data-backticks="1" data-nodeid="3696">position</code>样式属性值为<code data-backticks="1" data-nodeid="3698">fixed</code>时，元素脱离文档流、且不占位，此时看上去与<code data-backticks="1" data-nodeid="3700">absolute</code>相似。但当我们进行<a href="https://about-position-1255459943.file.myqcloud.com/position-fixed-absolute.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="3704">页面</a>滚动的时候，会发现<code data-backticks="1" data-nodeid="3706">fixed</code>元素位置没有发生变化。</p>
<p data-nodeid="3259"><img src="https://s0.lgstatic.com/i/image6/M01/33/F8/Cgp9HWBwB7uAbshiAADWWsHZghM208.png" alt="Drawing 22.png" data-nodeid="3711"></p>
<div data-nodeid="3260"><p style="text-align:center">（fixed 元素同样不占位）</p></div>
<p data-nodeid="3261">这是因为<code data-backticks="1" data-nodeid="3713">fixed</code>元素相对于浏览器窗口进行定位，而<code data-backticks="1" data-nodeid="3715">absolute</code>元素只有在满足“无<code data-backticks="1" data-nodeid="3717">static</code>定位以外的父元素”的时候，才会相对于<code data-backticks="1" data-nodeid="3719">document</code>进行定位。</p>
<p data-nodeid="3262">回到问题 3，将一个元素固定在页面的某个位置，可以通过给元素或是其父类元素添加<code data-backticks="1" data-nodeid="3722">position: fixed</code>或者<code data-backticks="1" data-nodeid="3724">position: absolute</code>将其固定在浏览器窗口或是文档页面中。</p>
<p data-nodeid="3263">使用元素定位可以将某个元素固定，那么同一个位置中存在多个元素的时候，就会发生元素的堆叠。</p>
<h4 data-nodeid="3264">元素堆叠 z-index</h4>
<p data-nodeid="3265">元素的堆叠方式和顺序，除了与<code data-backticks="1" data-nodeid="3729">position</code>定位有关，也与<code data-backticks="1" data-nodeid="3731">z-index</code>有关。通过设置<code data-backticks="1" data-nodeid="3733">z-index</code>值，我们可以设置元素的堆叠顺序，比如我们给同级的元素添加<code data-backticks="1" data-nodeid="3735">z-index值</code>：</p>
<p data-nodeid="3266"><img src="https://s0.lgstatic.com/i/image6/M00/33/F8/Cgp9HWBwB9GAVImUAAC4RJoX2o8350.png" alt="Drawing 24.png" data-nodeid="3739"></p>
<div data-nodeid="3267"><p style="text-align:center">（z-index 可改变元素堆叠顺序）</p></div>
<p data-nodeid="3268">在<a href="https://about-position-1255459943.file.myqcloud.com/position-z-index-same-level.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="3743">浏览器</a>中，我们可以看到：</p>
<ul data-nodeid="3269">
<li data-nodeid="3270">
<p data-nodeid="3271">当同级元素不设置<code data-backticks="1" data-nodeid="3746">z-index</code>或者<code data-backticks="1" data-nodeid="3748">z-index</code>相等时，后面的元素会叠在前面的元素上方；</p>
</li>
<li data-nodeid="3272">
<p data-nodeid="3273">当同级元素<code data-backticks="1" data-nodeid="3751">z-index</code>不同时，<code data-backticks="1" data-nodeid="3753">z-index</code>大的元素会叠在<code data-backticks="1" data-nodeid="3755">z-index</code>小的元素上方。</p>
</li>
</ul>
<p data-nodeid="3274"><code data-backticks="1" data-nodeid="3757">z-index</code>样式属性比较常用于多个元素层级控制的时候，比如弹窗一般需要在最上层，就可以通过设置较大的<code data-backticks="1" data-nodeid="3759">z-index</code>值来控制。</p>
<p data-nodeid="3275">那么，我们来看问题 4： 为什么将某个元素<code data-backticks="1" data-nodeid="3762">z-index</code>设置为<code data-backticks="1" data-nodeid="3764">9999999</code>，但是它依然被其他元素遮挡住了呢？</p>
<p data-nodeid="3276">这是因为除了同级元素以外，<code data-backticks="1" data-nodeid="3767">z-index</code>值的设置效果还会受到父元素的<code data-backticks="1" data-nodeid="3769">z-index</code>值的影响。<code data-backticks="1" data-nodeid="3771">z-index</code>值的设置只决定同一父元素中的同级子元素的堆叠顺序。因此，即使将某个元素<code data-backticks="1" data-nodeid="3773">z-index</code>设置为<code data-backticks="1" data-nodeid="3775">9999999</code>，它依然可能因为父元素的<code data-backticks="1" data-nodeid="3777">z-index</code>值小于其他父元素同级的元素，而导致该元素依然被其他元素遮挡。</p>
<p data-nodeid="3277">现在，我们解答了问题 1~4，同时还学习了关于 CSS 页面布局的核心规则，包括：</p>
<ul data-nodeid="3278">
<li data-nodeid="3279">
<p data-nodeid="3280">盒模型主要用来描述元素所占空间的内容；</p>
</li>
<li data-nodeid="3281">
<p data-nodeid="3282">一个元素属于内联元素还是块状元素，会影响它是否可以和其他元素位于同一行、宽高设置是否有效；</p>
</li>
<li data-nodeid="3283">
<p data-nodeid="3284">正常的文档流在 HTML 里面为从上到下、从左到右的排版布局，使用<code data-backticks="1" data-nodeid="3785">position</code>属性可以使元素脱离正常的文档流；</p>
</li>
<li data-nodeid="3285">
<p data-nodeid="3286">使用<code data-backticks="1" data-nodeid="3788">z-index</code>属性可以设置元素的堆叠顺序。</p>
</li>
</ul>
<p data-nodeid="3287">掌握了这些页面布局的规则，可以解决我们日常页面中单个元素样式调整中的大多数问题。对于进行整体的页面布局，比如设置元素居中、排版、区域划分等，涉及多个元素的布局，这种情况下常常会用到 Flex、Grid 这样的页面布局方式。下面我们一起来看看。</p>
<h3 data-nodeid="3288">常见页面布局方式</h3>
<p data-nodeid="3289">在我们的日常工作中，实现页面的 UI 样式除了会遇到单个元素的样式调整外，还需要对整个页面进行结构布局，比如将页面划分为左中右、上中下模块，实现某些模块的居中对齐，实现页面的响应式布局，等等。</p>
<p data-nodeid="3290">要实现对页面的排版布局，需要使用到一些页面布局方式。目前来说，比较常见的布局方式主要有三种：</p>
<ul data-nodeid="3291">
<li data-nodeid="3292">
<p data-nodeid="3293">传统布局方式；</p>
</li>
<li data-nodeid="3294">
<p data-nodeid="3295">Flex 布局方式；</p>
</li>
<li data-nodeid="3296">
<p data-nodeid="3297">Grid 布局方式。</p>
</li>
</ul>
<h4 data-nodeid="3298">传统布局</h4>
<p data-nodeid="3299">传统布局方式基本上使用上面介绍的布局规则，结合<code data-backticks="1" data-nodeid="3799">display</code>/<code data-backticks="1" data-nodeid="3801">position</code>/<code data-backticks="1" data-nodeid="3803">float</code>属性以及一些边距、x/y 轴距离等方式来进行布局。</p>
<p data-nodeid="3300">除了使用<code data-backticks="1" data-nodeid="3806">position: fixed</code>或者<code data-backticks="1" data-nodeid="3808">position: absolute</code>时，会使元素脱离文档流，使用<code data-backticks="1" data-nodeid="3810">float</code>属性同样会导致元素脱离文档流。</p>
<p data-nodeid="3301">这就涉及问题 5：为什么将某个元素里面的元素设置为<code data-backticks="1" data-nodeid="3813">float</code>之后，这个元素的高度就歪了呢？</p>
<p data-nodeid="3302">这是因为当我们给元素的<code data-backticks="1" data-nodeid="3816">float</code>属性赋值后，元素会脱离文档流，进行左右浮动，比如这里我们将其中一个<code data-backticks="1" data-nodeid="3818">&lt;div&gt;</code>元素添加了<code data-backticks="1" data-nodeid="3820">float</code>属性 ：</p>
<pre class="lang-js" data-nodeid="3303"><code data-language="js">&lt;style&gt;
  div {
    <span class="hljs-attr">border</span>: solid <span class="hljs-number">1</span>px red;
    width: <span class="hljs-number">50</span>px;
    height: <span class="hljs-number">50</span>px;
  }
  .float {
    <span class="hljs-attr">float</span>: left;
  }
&lt;/style&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>1<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"float"</span>&gt;</span>2<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"float"</span>&gt;</span>3<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>4<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>5<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"float"</span>&gt;</span>6<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
</code></pre>
<p data-nodeid="3304">我们可以在<a href="https://about-position-1255459943.file.myqcloud.com/display-float.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="3825">浏览器</a>中看到，<code data-backticks="1" data-nodeid="3827">float</code>元素会紧贴着父元素或者是上一个同级同浮动元素的边框：</p>
<p data-nodeid="3305"><img src="https://s0.lgstatic.com/i/image6/M01/33/F8/Cgp9HWBwB-SAZahpAABKURkJ8hE997.png" alt="Drawing 26.png" data-nodeid="3831"></p>
<p data-nodeid="3306">可以看到当元素设置为<code data-backticks="1" data-nodeid="3833">float</code>之后，它就脱离文档流，同时也不再占据原本的空间。</p>
<p data-nodeid="3307">因此，问题 5 的答案为：本属于普通流中的元素浮动之后，父元素内部如果不存在其他普通流元素了，就会表现出高度为 0，又称为高度塌陷。</p>
<p data-nodeid="3308">在这样的情况下，我们可以使用以下方法撑开父元素：</p>
<ul data-nodeid="3309">
<li data-nodeid="3310">
<p data-nodeid="3311">父元素使用<code data-backticks="1" data-nodeid="3838">overflow: hidden</code>（此时高度为<code data-backticks="1" data-nodeid="3840">auto</code>）；</p>
</li>
<li data-nodeid="3312">
<p data-nodeid="3313" class="">使父元素也成为浮动<code data-backticks="1" data-nodeid="3843">float</code>元素；</p>
</li>
<li data-nodeid="3314">
<p data-nodeid="3315" class="">使用<code data-backticks="1" data-nodeid="3846">clear</code>清除浮动。</p>
</li>
</ul>
<p data-nodeid="57081" class="te-preview-highlight">除了 <code data-backticks="1" data-nodeid="57083">clear</code> 清除浮动之外，这些方法为什么可以达到撑开父元素的效果呢，这是因为 BFC（Block Formatting Context，块格式化上下文）的特性。BFC 是 Web 页面的可视 CSS 渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域，详情大家可以私下了解下。</p>



































<p data-nodeid="3317">传统方式布局的优势在于兼容性较好，在一些版本较低的浏览器上也能给到用户较友好的体验。但传统布局需要掌握的知识较多也相对复杂，对于整个页面的布局和排版实现，常常是基于盒模型、使用<code data-backticks="1" data-nodeid="3850">display</code>属性+<code data-backticks="1" data-nodeid="3852">position</code>属性+<code data-backticks="1" data-nodeid="3854">float</code>属性的方式来进行，这个过程比较烦琐，因此更多时候我们都会使用开源库（比如 bootstrap）来完成页面布局。</p>
<p data-nodeid="3318">后来 W3C 提出了新的布局方式，可以快速、简便地实现页面的排版布局，新的布局方式包括 Flex 布局和 Grid 布局。</p>
<h4 data-nodeid="3319">使用 Flex 布局</h4>
<p data-nodeid="3320">Flex 布局（又称为 flexbox）是一种一维的布局模型。在使用此布局时，需掌握几个概念。</p>
<ol data-nodeid="3321">
<li data-nodeid="3322">
<p data-nodeid="3323">flexbox 的两根轴线。其中，主轴由<code data-backticks="1" data-nodeid="3860">flex-direction</code>定义，交叉轴则垂直于主轴。</p>
</li>
<li data-nodeid="3324">
<p data-nodeid="3325">在 flexbox 中，使用起始和终止来描述布局方向。</p>
</li>
<li data-nodeid="3326">
<p data-nodeid="3327">认识 flex 容器和 flex 元素。</p>
</li>
</ol>
<p data-nodeid="3328">想熟练使用 Flex 布局，我们需要了解什么是 flex 容器和 flex 元素。比如我们给一个父元素<code data-backticks="1" data-nodeid="3865">div</code>设置<code data-backticks="1" data-nodeid="3867">display: flex;</code>：</p>
<pre class="lang-js" data-nodeid="3329"><code data-language="js">&lt;style&gt;
  div {
    border: solid 1px #000;
    margin: 10px;
  }
  .box {
    display: flex;
  }
&lt;/style&gt;
&lt;div class="box"&gt;
  &lt;div&gt;1&lt;/div&gt;
  &lt;div&gt;2&lt;/div&gt;
  &lt;div&gt;3 &lt;br /&gt;有其他 &lt;br /&gt;内容&lt;/div&gt;
&lt;/div&gt;
</code></pre>
<p data-nodeid="3330">在浏览器中的效果就会如图所示：</p>
<p data-nodeid="3331"><img src="https://s0.lgstatic.com/i/image6/M01/34/01/CioPOWBwB_WAR3CJAAAti6zxREI918.png" alt="Drawing 28.png" data-nodeid="3872"></p>
<p data-nodeid="3332">其中，flex 容器为<code data-backticks="1" data-nodeid="3874">&lt;div class="box"&gt;</code>元素及其内部区域，而容器的直系子元素（1、2、3 这 3 个<code data-backticks="1" data-nodeid="3876">&lt;div&gt;</code>）为 flex 元素。</p>
<p data-nodeid="3333">在掌握了 flex 容器和 flex 元素之后，我们就可以通过调整 flexbox 轴线方向、排列方向和对齐方式的方式，实现需要的页面效果。</p>
<p data-nodeid="3334">Flex 布局种常用的方式包括：</p>
<ul data-nodeid="3335">
<li data-nodeid="3336">
<p data-nodeid="3337">通过<code data-backticks="1" data-nodeid="3881">flex-direction</code>调整 Flex 元素的排列方向（主轴的方向）；</p>
</li>
<li data-nodeid="3338">
<p data-nodeid="3339">用<code data-backticks="1" data-nodeid="3884">flex-wrap</code>实现多行 Flex 容器如何换行；</p>
</li>
<li data-nodeid="3340">
<p data-nodeid="3341">使用<code data-backticks="1" data-nodeid="3887">justify-content</code>调整 Flex 元素在主轴上的对齐方式；</p>
</li>
<li data-nodeid="3342">
<p data-nodeid="3343">使用<code data-backticks="1" data-nodeid="3890">align-items</code>调整 Flex 元素在交叉轴上如何对齐；</p>
</li>
<li data-nodeid="3344">
<p data-nodeid="3345">使用<code data-backticks="1" data-nodeid="3893">align-content</code>调整多根轴线的对齐方式。</p>
</li>
</ul>
<p data-nodeid="3346">Flex 布局给<code data-backticks="1" data-nodeid="3896">flexbox</code>的子元素之间提供了强大的空间分布和对齐能力，我们可以方便地使用 Flex 布局来实现垂直和水平居中，比如通过将元素设置为<code data-backticks="1" data-nodeid="3898">display: flex;</code>，并配合使用<code data-backticks="1" data-nodeid="3900">align-items: center;</code>、<code data-backticks="1" data-nodeid="3902">justify-content: center;</code>：</p>
<pre class="lang-js" data-nodeid="3347"><code data-language="js">&lt;style&gt;
  div {
    border: solid 1px #000;
  }
  .box {
    display: flex;
    width: 200px;
    height: 200px;
    align-items: center;
    justify-content: center;
  }
  .in-box {
    width: 80px;
    height: 80px;
  }
&lt;/style&gt;
&lt;div class="box"&gt;
  &lt;div class="in-box"&gt;我想要垂直水平居中&lt;/div&gt;
&lt;/div&gt;
</code></pre>
<p data-nodeid="3348">就可以将一个元素设置为垂直和水平居中：</p>
<p data-nodeid="3349"><img src="https://s0.lgstatic.com/i/image6/M01/34/01/CioPOWBwCA-Ad4tQAAA-HOY2i2w574.png" alt="Drawing 30.png" data-nodeid="3907"></p>
<p data-nodeid="3350">对于传统的布局方式来说，要实现上述垂直水平居中，常常需要依赖绝对定位+元素偏移的方式来实现，该实现方式不够灵活（在调整元素大小时需要调整定位）、难以维护。</p>
<p data-nodeid="3351">Flex 布局的出现，解决了很多前端开发居中、排版的一些痛点，尤其是垂直居中，因此现在几乎成为主流的布局方式。除此之外，还可以对 Flex 元素设置排列顺序、放大比例、缩小比例等。</p>
<p data-nodeid="3352">如果说 Flex 布局是一维布局，那么 Grid 布局则是一种二维布局的方式。</p>
<h4 data-nodeid="3353">Grid 布局</h4>
<p data-nodeid="3354">Grid 布局又称为网格布局，它将一个页面划分为几个主要区域，以及定义这些区域的大小、位置、层次等关系。</p>
<p data-nodeid="3355">我们知道 Flex 布局是基于轴线布局，与之相对，Grid 布局则是将容器划分成行和列，可以像表格一样按行或列来对齐元素。</p>
<p data-nodeid="3356">对于 Grid 布局，同样需要理解几个概念：网格轨道与行列、网格线、网格容器等。其实 Grid 布局很多概念跟 Flex 布局还挺相似的，因此这里不再赘述。</p>
<p data-nodeid="3357">使用 Grid 布局可以：</p>
<ul data-nodeid="3358">
<li data-nodeid="3359">
<p data-nodeid="3360">实现网页的响应式布局；</p>
</li>
<li data-nodeid="3361">
<p data-nodeid="3362">实现灵活的 12 列布局（类似于 Bootstrap 的 CSS 布局方式）；</p>
</li>
<li data-nodeid="3363">
<p data-nodeid="3364">与其他布局方式结合，与 css 其他部分协同合作。</p>
</li>
</ul>
<p data-nodeid="3365">通过 Grid 布局我们能实现任意组合不同布局，其设计可称得上目前最强大的布局方式，它与 Flex 布局是未来的趋势。其中，Grid 布局适用于较大规模的布局，Flex 布局则适合页面中的组件和较小规模布局。</p>
<h3 data-nodeid="3366">小结</h3>
<p data-nodeid="3367">今天我带大家学习了页面布局中比较核心的一些规则，包括盒模型计算、内联元素与块状元素布局规则、文档流布局和元素堆叠顺序。我们在写 CSS 过程中会遇到很多的“神奇”现象，而要理解这些现象并解决问题，掌握这些页面布局的原理逻辑和规则很重要。</p>
<p data-nodeid="3368">除了页面布局规则之外，我还带大家认识了常见的页面布局方式，包括传统布局方式、FleX 布局和 Grid 布局。</p>
<p data-nodeid="3369">细心的你或许也发现了，我们还遗留了问题 6 没有给出具体的答案：让一个元素进行垂直和水平居中，有多少种实现方式？</p>
<p data-nodeid="3370" class="">这个问题，我希望你可以自己进行解答，欢迎你将答案写在留言区～</p>

---

### 精选评论

##### **飞：
> 方法挺多的，我常用的主要是如下三种1. 知道元素宽高：margin: 0 auto; position: relative; top: 50%; margin-top: -1/2元素高度2. 不知道元素宽高：margin: 0 auto; position:relative; top:50%;transform: translateY(-50%)3. display: flex;align-items:center;justify-content:center;

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 赞哦

##### **宇：
> 传统布局那一块 div4 和 div5重合是为什么呢。。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 其实你在浏览器中进行观察，就会发现实际上 div4 和 div5 的占位并没有重合，如果你试试给它们加上背景颜色就能看的比较清楚了。之所以看起来它们重合了，是因为浮动元素会对相邻的元素造成影响，其中就包括了文字会尽可能围绕浮动元素。在 div4 上加上 clear: left; 就可以清除 float 带来的影响

##### Change：
> 垂直居中的方式：1、通过 inline-block设置元素 height和 line-height 、text-align:center 来进行垂直居中。2、通过 position:absoulte 加偏移量3、通过 flex布局属性 justity-content:center; align-items:center;

##### *聪：
> position还有一个可取值：sticky

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你说的没错，sticky 可以进行粘性定位，在页面滚动的过程中很有用，是我这边写漏了

##### **涛：
> 之前快手的面试官和我聊到：浮动的元素会不会脱离文档流，我的答案是会，但是面试官告诉我不会。文章里也提到了这一点，所以来讨论一下，就如这个问题中（https://segmentfault.com/q/1010000002870442/a-1020000002870502）提到的一样：float元素并不会彻底地脱离文档流

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以查看 MDN 中的描述：“float CSS属性指定一个元素应沿其容器的左侧或右侧放置，允许文本和内联元素环绕它。该元素从网页的正常流动(文档流)中移除，尽管仍然保持部分的流动性。”可见，浮动元素的确会从文档流中删除，但它并不是完全彻底地移除。
而我们常说的“脱离”二字并没有官方的定义，因此你和你的面试官理解或许不一样。如果将脱离认为是不符合正常的文档流，那么它便是脱离的；如果将脱离认为是彻底从文档流中移除，那么它便是不会脱离的。

##### **圳：
> 九种方式，定宽高3种，不定宽高6种

##### **北：
> 疑惑，float布局部分，div6也设置了float，按理说应该会向上同级浮动元素，不应该是div4嘛，为啥4.5还重叠了，6位置还是之前的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 其实你在浏览器中进行观察，就会发现实际上 div4 和 div5 的占位并没有重合，如果你试试给它们加上背景颜色就能看的比较清楚了。之所以看起来它们重合了，是因为浮动元素会对相邻的元素造成影响，其中就包括了文字会尽可能围绕浮动元素。在 div4 上加上 clear: left; 就可以清除 float 带来的影响

##### **星：
> 老师，float布局这里我有一个疑问，div2, div3 设置了float:left, 脱离文档流，div2重叠到div4上面，为什么4被挤出了盒子和5重叠了而不是2与4重叠呢，我知道实际上是div2和div4重叠的，不理解的是为什么div4里边的文字4被挤了出去

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为浮动元素会对相邻的元素造成影响，其中就包括了文字会尽可能围绕浮动元素。在 div4 上加上 clear: left; 就可以清除 float 带来的影响

##### *雅：
> **飞的第一二两种方式，position:relative配top: 50%真的可以吗？试了不行啊，position:absolute时，top:50%才起效吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，应该是父元素需要将 position 设置为 relative/absolute/fixed，同时子元素需要为 position:absolute

##### *好：
> 除了前面的还有tabel-cell

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，不过 table-cell 现在很少人使用了，flexbox 使用要方便和简单很多

##### **星：
> 都是干货😀

##### *聪：
> 纠正老师的读音：赘（zhuì）述😀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; [哭泣]我的确想这么念的来着，可是我的广普不允许

##### **贤：
> 使用display:table-cell，还有一种就是使用grid网格行列去实现

