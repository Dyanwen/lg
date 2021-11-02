<p data-nodeid="43420">在本模块中，我为你设计了一个基于真实数据的项目，也就是前面提到的商业智能系统，可以让你对 Spark 有更形象的认识，并对前面学习的知识进行巩固。为了便于理解，该模块简化了业务复杂度和技术复杂度，作为你的第一个 Spark 项目是比较合适的。</p>
<p data-nodeid="43421">本课时的主要内容是：</p>
<ul data-nodeid="43422">
<li data-nodeid="43423">
<p data-nodeid="43424">美国的大众点评网 Yelp 和 Yelp 2016 Dataset Challenge 数据集介绍。</p>
</li>
<li data-nodeid="43425">
<p data-nodeid="43426">作为 Yelp 运营负责人，你希望知道什么？</p>
</li>
</ul>
<h3 data-nodeid="43427">Yelp 与数据集介绍</h3>
<p data-nodeid="43428">Yelp 由前 PayPal 员工罗素·西蒙斯和杰里米·斯托普尔曼于 2004 年在美国旧金山成立，公司发展迅速，得到几轮融资支持，2010 年营业额达到 3 千万美元，以及 450 万评论量。2009 年到 2012 年，其在欧洲和亚洲的业务也蓬勃发展。</p>
<p data-nodeid="43429">这是一个著名的商户点评网站，与我们的大众点评网类似，美国各地用户只要注册就能在 Yelp 网站上给商户打分、评论，还能和其他用户交流感想体验。Yelp 上的商户包括各地餐馆、购物中心、酒店、旅游等。</p>
<p data-nodeid="43430">就像 Yelp 的 slogan 一样，这家公司看重的是真实客户的真实评价，尤其注重把一小部分热衷点评的用户吸引过来，同时给予优质客户奖励。这种形式提高了 Yelp 网站上各种信息的可信度，也把有深入体验的优质用户和那些走马观花的用户区分开了。</p>
<p data-nodeid="43431">Yelp 在 2016 年公开了其内部业务数据集（Yelp 2016 Dataset Challenge），供数据科学竞赛爱好者使用，Kaggle 也收录了该数据集（下载地址：<a href="https://www.kaggle.com/yelp-dataset/yelp-dataset" data-nodeid="43496">https://www.kaggle.com/yelp-dataset/yelp-dataset</a>）。由于是真实数据集，其中有不少有趣的内容，我们这次的项目也是基于该数据集。</p>
<p data-nodeid="43432">下面我们对数据集进行一个介绍，它包括以下几个表：</p>
<ul data-nodeid="43433">
<li data-nodeid="43434">
<p data-nodeid="43435">业务表 (businesss)</p>
</li>
<li data-nodeid="43436">
<p data-nodeid="43437">评价表 (reviews)</p>
</li>
<li data-nodeid="43438">
<p data-nodeid="43439">小贴士表 (tips)</p>
</li>
<li data-nodeid="43440">
<p data-nodeid="43441">用户信息表 (user information)</p>
</li>
<li data-nodeid="43442">
<p data-nodeid="43443">签到表 (check-ins)</p>
</li>
</ul>
<p data-nodeid="43444">数据集共 10G 左右，包含 668 万条评论。它来自 19 万个商业机构，涵盖了 10 个都市区域，有大概 100 多万个属性标签。此外，Yelp 还提供了一些图像数据，比如商户的图片数据，由于在本项目中用不到，故不对其进行介绍。</p>
<h4 data-nodeid="43445">business 表</h4>
<p data-nodeid="43446">以下代码是以 Json 格式表示的 business 表中的一条数据，代表了一个商业组织的相关数据（例如某个餐馆）。</p>
<pre class="lang-java" data-nodeid="45444"><code data-language="java">{
    <span class="hljs-comment">// string, 22 character unique string business id</span>
    <span class="hljs-string">"business_id"</span>: <span class="hljs-string">"tnhfDv5Il8EaGSXZGiuQGg"</span>,
    <span class="hljs-comment">// string, the business's name</span>
    <span class="hljs-string">"name"</span>: <span class="hljs-string">"Garaje"</span>,
    <span class="hljs-comment">// string, the full address of the business</span>
    <span class="hljs-string">"address"</span>: <span class="hljs-string">"475 3rd St"</span>,
    <span class="hljs-comment">// string, the city</span>
    <span class="hljs-string">"city"</span>: <span class="hljs-string">"San Francisco"</span>,
    <span class="hljs-comment">// string, 2 character state code, if applicable</span>
    <span class="hljs-string">"state"</span>: <span class="hljs-string">"CA"</span>,
    <span class="hljs-comment">// string, the postal code</span>
    <span class="hljs-string">"postal code"</span>: <span class="hljs-string">"94107"</span>,
    <span class="hljs-comment">// float, latitude</span>
    <span class="hljs-string">"latitude"</span>: <span class="hljs-number">37.7817529521</span>,
    <span class="hljs-comment">// float, longitude</span>
    <span class="hljs-string">"longitude"</span>: -<span class="hljs-number">122.39612197</span>,
    <span class="hljs-comment">// float, star rating, rounded to half-stars</span>
    <span class="hljs-string">"stars"</span>: <span class="hljs-number">4.5</span>,
    <span class="hljs-comment">// integer, number of reviews</span>
    <span class="hljs-string">"review_count"</span>: <span class="hljs-number">1198</span>,
    <span class="hljs-comment">// integer, 0 or 1 for closed or open, respectively</span>
    <span class="hljs-string">"is_open"</span>: <span class="hljs-number">1</span>,
    <span class="hljs-comment">// object, business attributes to values. note: some attribute values might be objects</span>
    <span class="hljs-string">"attributes"</span>: {
        <span class="hljs-string">"RestaurantsTakeOut"</span>: <span class="hljs-keyword">true</span>,
        <span class="hljs-string">"BusinessParking"</span>: {
            <span class="hljs-string">"garage"</span>: <span class="hljs-keyword">false</span>,
            <span class="hljs-string">"street"</span>: <span class="hljs-keyword">true</span>,
            <span class="hljs-string">"validated"</span>: <span class="hljs-keyword">false</span>,
            <span class="hljs-string">"lot"</span>: <span class="hljs-keyword">false</span>,
            <span class="hljs-string">"valet"</span>: <span class="hljs-keyword">false</span>
        },
    },
    <span class="hljs-comment">// an array of strings of business categories</span>
    <span class="hljs-string">"categories"</span>: [
        <span class="hljs-string">"Mexican"</span>,
        <span class="hljs-string">"Burgers"</span>,
        <span class="hljs-string">"Gastropubs"</span>
    ],
    <span class="hljs-comment">// an object of key day to value hours, hours are using a 24hr clock</span>
    <span class="hljs-string">"hours"</span>: {
        <span class="hljs-string">"Monday"</span>: <span class="hljs-string">"10:00-21:00"</span>,
        <span class="hljs-string">"Tuesday"</span>: <span class="hljs-string">"10:00-21:00"</span>,
        <span class="hljs-string">"Friday"</span>: <span class="hljs-string">"10:00-21:00"</span>,
        <span class="hljs-string">"Wednesday"</span>: <span class="hljs-string">"10:00-21:00"</span>,
        <span class="hljs-string">"Thursday"</span>: <span class="hljs-string">"10:00-21:00"</span>,
        <span class="hljs-string">"Sunday"</span>: <span class="hljs-string">"11:00-18:00"</span>,
        <span class="hljs-string">"Saturday"</span>: <span class="hljs-string">"10:00-21:00"</span>
    }
}
</code></pre>




<p data-nodeid="45959">为了让你对这些字段有更直观的认识，我截取了 Yelp 网站商家主页的部分内容，如下图所示，网页内容可以很容易地和上述数据对应。</p>
<p data-nodeid="46356" class=""><img src="https://s0.lgstatic.com/i/image/M00/46/50/Ciqc1F9E02aAJy1jAAOsbGl7FgA043.png" alt="Drawing 0.png" data-nodeid="46359"><br>
<img src="https://s0.lgstatic.com/i/image/M00/46/50/Ciqc1F9E02-AYCSuAACY_JFFhNA737.png" alt="Drawing 1.png" data-nodeid="46363"></p>




<p data-nodeid="43450">结合网页内容，我们可以更形象地观察上面的代码，它包含商家的开业时间、评论条数、地理位置，以及属性值与类目信息等。</p>
<h4 data-nodeid="43451">review 表</h4>
<p data-nodeid="43452">以下代码是以 Json  格式表示的 review 表中的一条数据，代表了一个用户（user）对一个商业机构（busniess）的一条评论。</p>
<pre class="lang-java" data-nodeid="46630"><code data-language="java">{
    <span class="hljs-comment">// string, 22 character unique review id</span>
    <span class="hljs-string">"review_id"</span>: <span class="hljs-string">"zdSx_SD6obEhz9VrW9uAWA"</span>,
    <span class="hljs-comment">// string, 22 character unique user id, maps to the user in user.json</span>
    <span class="hljs-string">"user_id"</span>: <span class="hljs-string">"Ha3iJu77CxlrFm-vQRs_8g"</span>,
    <span class="hljs-comment">// string, 22 character business id, maps to business in business.json</span>
    <span class="hljs-string">"business_id"</span>: <span class="hljs-string">"tnhfDv5Il8EaGSXZGiuQGg"</span>,
    <span class="hljs-comment">// integer, star rating</span>
    <span class="hljs-string">"stars"</span>: <span class="hljs-number">4</span>,
    <span class="hljs-comment">// string, date formatted YYYY-MM-DD</span>
    <span class="hljs-string">"date"</span>: <span class="hljs-string">"2016-03-09"</span>,
    <span class="hljs-comment">// string, the review itself</span>
    <span class="hljs-string">"text"</span>: <span class="hljs-string">"Great place to hang out after work: the prices are decent, and the ambience is fun. It's a bit loud, but very lively. The staff is friendly, and the food is good. They have a good selection of drinks."</span>,
    <span class="hljs-comment">// integer, number of useful votes received</span>
    <span class="hljs-string">"useful"</span>: <span class="hljs-number">0</span>,
    <span class="hljs-comment">// integer, number of funny votes received</span>
    <span class="hljs-string">"funny"</span>: <span class="hljs-number">0</span>,
    <span class="hljs-comment">// integer, number of cool votes received</span>
    <span class="hljs-string">"cool"</span>: <span class="hljs-number">0</span>
}
</code></pre>

<p data-nodeid="47155">结合网页中与评论相关的内容进行观察，如下图所示：</p>
<p data-nodeid="47156" class=""><img src="https://s0.lgstatic.com/i/image/M00/46/50/Ciqc1F9E03uAV6oEAAVp7j4PaHc367.png" alt="Drawing 2.png" data-nodeid="47160"></p>


<p data-nodeid="43456">评论表的一条数据包含完整的评论文本数据，包括撰写评论的 user_id 和撰写评论的对象 business_id。结合网页，同样会让你有更形象的感知。</p>
<h4 data-nodeid="43457">user 表</h4>
<p data-nodeid="43458">以下代码是以 Json 格式表示的 user 表中的一条数据，代表了一个用户的基本情况。</p>
<pre class="lang-java" data-nodeid="47427"><code data-language="java">{
    <span class="hljs-comment">// string, 22 character unique user id, maps to the user in user.json</span>
    <span class="hljs-string">"user_id"</span>: <span class="hljs-string">"Ha3iJu77CxlrFm-vQRs_8g"</span>,
    <span class="hljs-comment">// string, the user's first name</span>
    <span class="hljs-string">"name"</span>: <span class="hljs-string">"Sebastien"</span>,
    <span class="hljs-comment">// integer, the number of reviews they've written</span>
    <span class="hljs-string">"review_count"</span>: <span class="hljs-number">56</span>,
    <span class="hljs-comment">// string, when the user joined Yelp, formatted like YYYY-MM-DD</span>
    <span class="hljs-string">"yelping_since"</span>: <span class="hljs-string">"2011-01-01"</span>,
    <span class="hljs-comment">// array of strings, an array of the user's friend as user_ids</span>
    <span class="hljs-string">"friends"</span>: [
        <span class="hljs-string">"wqoXYLWmpkEH0YvTmHBsJQ"</span>,
        <span class="hljs-string">"KUXLLiJGrjtSsapmxmpvTA"</span>,
        <span class="hljs-string">"6e9rJKQC3n0RSKyHLViL-Q"</span>
    ],
    <span class="hljs-comment">// integer, number of useful votes sent by the user</span>
    <span class="hljs-string">"useful"</span>: <span class="hljs-number">21</span>,
    <span class="hljs-comment">// integer, number of funny votes sent by the user</span>
    <span class="hljs-string">"funny"</span>: <span class="hljs-number">88</span>,
    <span class="hljs-comment">// integer, number of cool votes sent by the user</span>
    <span class="hljs-string">"cool"</span>: <span class="hljs-number">15</span>,
    <span class="hljs-comment">// integer, number of fans the user has</span>
    <span class="hljs-string">"fans"</span>: <span class="hljs-number">1032</span>,
    <span class="hljs-comment">// array of integers, the years the user was elite</span>
    <span class="hljs-string">"elite"</span>: [
        <span class="hljs-number">2012</span>,
        <span class="hljs-number">2013</span>
    ],
    <span class="hljs-comment">// float, average rating of all reviews</span>
    <span class="hljs-string">"average_stars"</span>: <span class="hljs-number">4.31</span>,
    <span class="hljs-comment">// integer, number of hot compliments received by the user</span>
    <span class="hljs-string">"compliment_hot"</span>: <span class="hljs-number">339</span>,
    <span class="hljs-comment">// integer, number of more compliments received by the user</span>
    <span class="hljs-string">"compliment_more"</span>: <span class="hljs-number">668</span>,
    <span class="hljs-comment">// integer, number of profile compliments received by the user</span>
    <span class="hljs-string">"compliment_profile"</span>: <span class="hljs-number">42</span>,
    <span class="hljs-comment">// integer, number of cute compliments received by the user</span>
    <span class="hljs-string">"compliment_cute"</span>: <span class="hljs-number">62</span>,
    <span class="hljs-comment">// integer, number of list compliments received by the user</span>
    <span class="hljs-string">"compliment_list"</span>: <span class="hljs-number">37</span>,
    <span class="hljs-comment">// integer, number of note compliments received by the user</span>
    <span class="hljs-string">"compliment_note"</span>: <span class="hljs-number">356</span>,
    <span class="hljs-comment">// integer, number of plain compliments received by the user</span>
    <span class="hljs-string">"compliment_plain"</span>: <span class="hljs-number">68</span>,
    <span class="hljs-comment">// integer, number of cool compliments received by the user</span>
    <span class="hljs-string">"compliment_cool"</span>: <span class="hljs-number">91</span>,
    <span class="hljs-comment">// integer, number of funny compliments received by the user</span>
    <span class="hljs-string">"compliment_funny"</span>: <span class="hljs-number">99</span>,
    <span class="hljs-comment">// integer, number of writer compliments received by the user</span>
    <span class="hljs-string">"compliment_writer"</span>: <span class="hljs-number">95</span>,
    <span class="hljs-comment">// integer, number of photo compliments received by the user</span>
    <span class="hljs-string">"compliment_photos"</span>: <span class="hljs-number">50</span>
}
</code></pre>

<p data-nodeid="47952">而与数据对应的网站用户主页部分如下图所示：</p>
<p data-nodeid="47953" class=""><img src="https://s0.lgstatic.com/i/image/M00/46/5B/CgqCHl9E04eAUECAAAK5xvD2DYI866.png" alt="Drawing 3.png" data-nodeid="47957"></p>


<p data-nodeid="43462">一条用户数据包括该用户的朋友以及与该用户关联的所有元数据。</p>
<h4 data-nodeid="43463">tip 表</h4>
<p data-nodeid="43464">以下代码是以 Json 格式表示的 tip 表中的一条数据，代表了一条简短的评论。</p>
<pre class="lang-java" data-nodeid="48224"><code data-language="java">{
    <span class="hljs-comment">// string, text of the tip</span>
    <span class="hljs-string">"text"</span>: <span class="hljs-string">"Secret menu - fried chicken sando is da bombbbbbb Their zapatos are good too."</span>,
    <span class="hljs-comment">// string, when the tip was written, formatted like YYYY-MM-DD</span>
    <span class="hljs-string">"date"</span>: <span class="hljs-string">"2013-09-20"</span>,
    <span class="hljs-comment">// integer, how many compliments it has</span>
    <span class="hljs-string">"compliment_count"</span>: <span class="hljs-number">172</span>,
    <span class="hljs-comment">// string, 22 character business id, maps to business in business.json</span>
    <span class="hljs-string">"business_id"</span>: <span class="hljs-string">"tnhfDv5Il8EaGSXZGiuQGg"</span>,
    <span class="hljs-comment">// string, 22 character unique user id, maps to the user in user.json</span>
    <span class="hljs-string">"user_id"</span>: <span class="hljs-string">"49JhAJh8vSQ-vM4Aourl0g"</span>
}
</code></pre>

<p data-nodeid="48749">与数据对应的网站小贴士页如下图所示：</p>
<p data-nodeid="48750" class=""><img src="https://s0.lgstatic.com/i/image/M00/46/50/Ciqc1F9E05KACWWQAAF_7xDrN9M074.png" alt="Drawing 4.png" data-nodeid="48754"></p>


<p data-nodeid="43468">这种小贴士的对象同样是商家，但小贴士比评论更简短，并且倾向于传达快速的建议。</p>
<h4 data-nodeid="43469">checkin 表</h4>
<p data-nodeid="43470">以下代码是以 Json 格式表示的 checkin 表中的一条数据，代表了一个商户的签到情况，包括商户的 business_id 及顾客的签到时间。</p>
<pre class="lang-java" data-nodeid="49021"><code data-language="java">{
    <span class="hljs-comment">// string, 22 character business id, maps to business in business.json</span>
    <span class="hljs-string">"business_id"</span>: <span class="hljs-string">"tnhfDv5Il8EaGSXZGiuQGg"</span>
    <span class="hljs-comment">// string which is a comma-separated list of timestamps for each checkin, each with format YYYY-MM-DD HH:MM:SS</span>
    <span class="hljs-string">"date"</span>: <span class="hljs-string">"2016-04-26 19:49:16, 2016-08-30 18:36:57, 2016-10-15 02:45:18, 2016-11-18 01:54:50, 2017-04-20 18:39:06, 2017-05-03 17:58:02"</span>
}
</code></pre>

<h3 data-nodeid="49288">从 Yelp 运营负责人的视角分析</h3>


<p data-nodeid="43474">高质量的评论是 Yelp 的灵魂，作为 Yelp 的运营负责人，你希望能从评论中发现些有趣的东西，从而更好地帮助商家，活跃用户。</p>
<p data-nodeid="43475">从数据方体的角度来看，review 表中 stars、useful、funny、cool 字段就是度量，而 busniess 表中的 city、state 字段以及 review 表中的 date 字段都可以作为时间与空间维度。通过数据方体与多维分析，能够很好地对数据进行分析。</p>
<p data-nodeid="43476">Yelp 的商业智能系统希望构建一个关于评论的数据方体，并进行相应的分析与可视化。通过这种方法，将得到以商业机构为主体的相关“业务知识”，以辅助运营人员更好地决策。例如评分更高的商业主体有哪些特征，以及有哪些特征是我们平时所忽略的，等等。详细方法在之后的课程会展开。</p>
<p data-nodeid="43477">这里注意，为了简化，本项目只会构建一个数据方体，但是在一般规模的生产环境中，商业智能系统需要构建的数据方体的数量非常多，基础数据也远远不止我们这个项目中的 5 张表。</p>
<p data-nodeid="43478"><strong data-nodeid="43553">需要特别说明的一点是</strong>，通常在数据仓库或者商业智能系统项目开始时，你并不会直接获取到上面下载的 Yelp 的数据集，我们需要的数据往往在业务数据库中，所以下个课时，我们将模拟将数据从业务数据库中导出的过程。</p>
<h3 data-nodeid="43835">总结</h3>


<p data-nodeid="43481">作为项目开始的第一个课时，我主要带你熟悉数据集、数据结构，了解商业智能系统的分析需求。为了便于理解，无论是对数据集还是分析需求都做了大量简化。如果是真实情况，每个环节都会增加上百倍的复杂性。所以近期课程都需要你反复巩固，以便更好地运用到工作中。</p>

---

### 精选评论


