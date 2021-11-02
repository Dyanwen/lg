<p>本课时我们进入实战课程的演练：探索葡萄牙银行电话调查的结果。本课时主要用真实数据构建了一个数据探索场景，这既不是一个项目也不是一个应用，只是一个探索的过程。</p>
<p>这个过程在实际应用中是非常常见的，无论是对分析师还是工程师来说，对数据的探索都是必要的，为此，Spark 也创新地开发了 Spark Shell 应用，让编译型语言 Scala 用起来像脚本语言一样，提高了实践效率。</p>
<p>数据（下载链接为：<a href="https://pan.baidu.com/s/1up25t-HQF16Sx4-naC0rJw">https://pan.baidu.com/s/1up25t-HQF16Sx4-naC0rJw</a>，密码为 jzke）来源于葡萄牙银行电话调查的结果，本课时会通过一些分析手段逐步对数据展开探索，直到用户得到想要的信息。本课时的内容完全是实践，没有理论，希望你动手一起做。由于案例中用到了 Dataset API，故采用 Scala 版本。</p>
<p>下面这段代码读取了葡萄牙银行通过电话访问进行市场调查得到的数据集，并统计了数据条数：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-keyword">import</span> org.apache.spark.sql.types._
<span class="hljs-keyword">import</span> org.apache.spark.sql.{<span class="hljs-type">SparkSession</span>}
<span class="hljs-keyword">import</span> org.apache.spark.sql.functions._
&nbsp;
<span class="hljs-keyword">case</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Call</span>(<span class="hljs-params">age: <span class="hljs-type">Double</span>, job: <span class="hljs-type">String</span>, marital: <span class="hljs-type">String</span>, edu: <span class="hljs-type">String</span>, 
credit_default: <span class="hljs-type">String</span>, housing: <span class="hljs-type">String</span>, loan: <span class="hljs-type">String</span>, 
contact: <span class="hljs-type">String</span>, month: <span class="hljs-type">String</span>, day: <span class="hljs-type">String</span>, 
dur: <span class="hljs-type">Double</span>, campaign: <span class="hljs-type">Double</span>, pdays: <span class="hljs-type">Double</span>, 
prev: <span class="hljs-type">Double</span>,pout: <span class="hljs-type">String</span>, emp_var_rate: <span class="hljs-type">Double</span>, 
cons_price_idx: <span class="hljs-type">Double</span>, cons_conf_idx: <span class="hljs-type">Double</span>, euribor3m: <span class="hljs-type">Double</span>, 
nr_employed: <span class="hljs-type">Double</span>, deposit: <span class="hljs-type">String</span></span>)</span>
&nbsp;
<span class="hljs-comment">//葡萄牙银行通过电话访问进行市场调查得到数据集，以下为21个字段</span>
<span class="hljs-comment">//受访者年龄</span>
<span class="hljs-keyword">val</span> age = <span class="hljs-type">StructField</span>(<span class="hljs-string">"age"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">IntegerType</span>)
<span class="hljs-comment">//受访者职业</span>
<span class="hljs-keyword">val</span> job = <span class="hljs-type">StructField</span>(<span class="hljs-string">"job"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//婚姻状态</span>
<span class="hljs-keyword">val</span> marital = <span class="hljs-type">StructField</span>(<span class="hljs-string">"marital"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//受教育程度</span>
<span class="hljs-keyword">val</span> edu = <span class="hljs-type">StructField</span>(<span class="hljs-string">"edu"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//是否信贷违约</span>
<span class="hljs-keyword">val</span> credit_default = <span class="hljs-type">StructField</span>(<span class="hljs-string">"credit_default"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//是否有房屋贷款</span>
<span class="hljs-keyword">val</span> housing = <span class="hljs-type">StructField</span>(<span class="hljs-string">"housing"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//是否有个人贷款</span>
<span class="hljs-keyword">val</span> loan = <span class="hljs-type">StructField</span>(<span class="hljs-string">"loan"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//联系类型（移动电话或座机）</span>
<span class="hljs-keyword">val</span> contact = <span class="hljs-type">StructField</span>(<span class="hljs-string">"contact"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//当天访谈的月份</span>
<span class="hljs-keyword">val</span> month = <span class="hljs-type">StructField</span>(<span class="hljs-string">"month"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//当天访谈时间的是星期几</span>
<span class="hljs-keyword">val</span> day = <span class="hljs-type">StructField</span>(<span class="hljs-string">"day"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//最后一次电话联系持续时间</span>
<span class="hljs-keyword">val</span> dur = <span class="hljs-type">StructField</span>(<span class="hljs-string">"dur"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">DoubleType</span>)
<span class="hljs-comment">//此次访谈的电话联系的次数</span>
<span class="hljs-keyword">val</span> campaign = <span class="hljs-type">StructField</span>(<span class="hljs-string">"campaign"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">DoubleType</span>)
<span class="hljs-comment">//距离早前访谈最后一次电话联系的天数</span>
<span class="hljs-keyword">val</span> pdays = <span class="hljs-type">StructField</span>(<span class="hljs-string">"pdays"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">DoubleType</span>)
<span class="hljs-comment">//早前访谈电话联系的次数</span>
<span class="hljs-keyword">val</span> prev = <span class="hljs-type">StructField</span>(<span class="hljs-string">"prev"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">DoubleType</span>)
<span class="hljs-comment">//早前访谈的结果，成功或失败</span>
<span class="hljs-keyword">val</span> pout = <span class="hljs-type">StructField</span>(<span class="hljs-string">"pout"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
<span class="hljs-comment">//就业变化率（季度指标）</span>
<span class="hljs-keyword">val</span> emp_var_rate = <span class="hljs-type">StructField</span>(<span class="hljs-string">"emp_var_rate"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">DoubleType</span>)
<span class="hljs-comment">//消费者物价指数（月度指标）</span>
<span class="hljs-keyword">val</span> cons_price_idx = <span class="hljs-type">StructField</span>(<span class="hljs-string">"cons_price_idx"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">DoubleType</span>)
<span class="hljs-comment">//消费者信心指数（月度指标）</span>
<span class="hljs-keyword">val</span> cons_conf_idx = <span class="hljs-type">StructField</span>(<span class="hljs-string">"cons_conf_idx"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">DoubleType</span>)
<span class="hljs-comment">//欧元银行间3月拆借率</span>
<span class="hljs-keyword">val</span> euribor3m = <span class="hljs-type">StructField</span>(<span class="hljs-string">"euribor3m"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">DoubleType</span>)
<span class="hljs-comment">//员工数量（季度指标）</span>
<span class="hljs-keyword">val</span> nr_employed = <span class="hljs-type">StructField</span>(<span class="hljs-string">"nr_employed"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">DoubleType</span>)
<span class="hljs-comment">//目标变量，是否会定期存款</span>
<span class="hljs-keyword">val</span> deposit = <span class="hljs-type">StructField</span>(<span class="hljs-string">"deposit"</span>, <span class="hljs-type">DataTypes</span>.<span class="hljs-type">StringType</span>)
&nbsp;
<span class="hljs-keyword">val</span> fields = <span class="hljs-type">Array</span>(age, job, marital, 
edu, credit_default, housing, 
loan, contact, month, 
day, dur, campaign, 
pdays, prev, pout, 
emp_var_rate, cons_price_idx, cons_conf_idx, 
euribor3m, nr_employed, deposit)
&nbsp;
<span class="hljs-keyword">val</span> schema = <span class="hljs-type">StructType</span>(fields)
&nbsp;
<span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>
.builder()
.appName(<span class="hljs-string">"data exploration"</span>)
.master(<span class="hljs-string">"local"</span>)
.getOrCreate()
<span class="hljs-keyword">import</span> spark.implicits._
&nbsp;
<span class="hljs-comment">//该数据集中的记录有些字段没用采集到数据为unknown</span>
<span class="hljs-keyword">val</span> df = spark
.read
.schema(schema)
.option(<span class="hljs-string">"sep"</span>, <span class="hljs-string">";"</span>)
.option(<span class="hljs-string">"header"</span>, <span class="hljs-literal">true</span>)
.csv(<span class="hljs-string">"./bank/bank-additional-full.csv"</span>)
&nbsp;
println(df.count())
</code></pre>
<p>运行之后的结果为：41188。接下来我们再来根据婚姻情况统计各类人群的数量和缺失值的数量。</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//该数据集将bank-additional-full.csv中原本是unknown的字段置为null</span>
<span class="hljs-keyword">val</span> dm = spark
.read
.schema(schema)
.option(<span class="hljs-string">"sep"</span>, <span class="hljs-string">";"</span>)
.option(<span class="hljs-string">"header"</span>, <span class="hljs-literal">true</span>)
.csv(<span class="hljs-string">"./bank/bank-additional-full-missing.csv"</span>)
<span class="hljs-comment">//根据婚姻情况统计各类人群的数量和缺失值的数量</span>
dm.groupBy(<span class="hljs-string">"marital"</span>).count().show()
</code></pre>
<p>结果为：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/CgqCHl7fP0eACZ5pAABAb_pyhCc901.png" alt="Drawing 0.png"></p>
<p>现在我们再根据职业统计各类人群的数量和缺失值的数量：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//根据职业统计各类人群的数量和缺失值的数量</span>
dm.groupBy(<span class="hljs-string">"job"</span>).count().show()
</code></pre>
<p>结果为：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/BE/Ciqc1F7fP1GADeUyAACYePuacpI978.png" alt="Drawing 1.png"></p>
<p>接下来根据教育情况统计各类人群的数量和缺失值的数量：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//根据教育情况统计各类人群的数量和缺失值的数量</span>
dm.groupBy(<span class="hljs-string">"edu"</span>).count().show()
</code></pre>
<p>结果为：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/BE/Ciqc1F7fP1mAS4AeAAB14vZ5jyI780.png" alt="Drawing 2.png"></p>
<p>下面我们选取数值类字段作为数据子集，进行描述性统计：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//选数值类字段作为数据子集，进行描述性统计（包括频次统计，平均值，标准差，最小值，最大值）</span>
<span class="hljs-keyword">val</span> dsSubset=dm.select(<span class="hljs-string">"age"</span>,<span class="hljs-string">"dur"</span>,<span class="hljs-string">"campaign"</span>,<span class="hljs-string">"prev"</span>,<span class="hljs-string">"deposit"</span>).cache()
<span class="hljs-comment">//通过描述性统计，可以对数据进行快速地检查。比如，频次统计可以检查数据的有效行数，年龄的平均值和范围可以判断数据样本是不是符合预期。通过均值和方差可以对数据进行更深入地分析，比如，假设数据服从正态分布，年龄的均值和标准差表明了受访者的年龄大多在30~50 之间。</span>
dsSubset.describe().show()
</code></pre>
<p>结果为：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/BE/Ciqc1F7fP2KAQ13MAACaLdyyyCM693.png" alt="Drawing 3.png"></p>
<p>下面这段代码判断了变量间的相关性：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//判断变量间相关性，计算变量间的协方差和相关系数，协方差表示两变量的变化方向相同或相反。age和dur的协方差为-2.3391469421265874，表示随着受访者的年龄增加，上一次访问时长减少。</span>
println(dsSubset.stat.cov(<span class="hljs-string">"age"</span>,<span class="hljs-string">"dur"</span>))
</code></pre>
<p>结果为：-2.3391469421265874。</p>
<p>接下来计算相关系数：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//相关系数（Pearson系数）表示变量间的相关程度。age和dur的相关系数为-8.657050101409117E-4，呈较弱的负相关性。</span>
println(dsSubset.stat.corr(<span class="hljs-string">"age"</span>,<span class="hljs-string">"dur"</span>))
</code></pre>
<p>结果为：8.657050101409117E-4。</p>
<p>下面计算每个年龄段的婚姻状态分布：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//交叉表,通过交叉表可以知道在每个年龄段的婚姻状态分布</span>
&nbsp;ds.stat.crosstab(<span class="hljs-string">"age"</span>,<span class="hljs-string">"marital"</span>).orderBy(<span class="hljs-string">"age_marital"</span>).show(<span class="hljs-number">20</span>)
</code></pre>
<p>结果为：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/CgqCHl7fP2uADq6rAADP-qyhVGc851.png" alt="Drawing 4.png"></p>
<p>下面这段代码展示了所有受访人的学历背景出现频率超过 0.3 的学历：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//所有受访人的学历背景出现频率超过0.3的学历</span>
println(ds.stat.freqItems(<span class="hljs-type">Seq</span>(<span class="hljs-string">"edu"</span>),<span class="hljs-number">0.3</span>).collect()(<span class="hljs-number">0</span>))
</code></pre>
<p>结果为：<br>
[WrappedArray(high.school, university.degree, professional.course)]。</p>
<p>下面计算受访用户年龄的分位数：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//四分位数,第三个参数0.0表示相对误差</span>
df.stat.approxQuantile(<span class="hljs-string">"age"</span>,<span class="hljs-type">Array</span>(<span class="hljs-number">0.25</span>,<span class="hljs-number">0.5</span>,<span class="hljs-number">0.75</span>),<span class="hljs-number">0.0</span>)
.foreach(println)
</code></pre>
<p>结果为：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/CgqCHl7fP3OALu9-AAAHMrnBXYU267.png" alt="Drawing 5.png"></p>
<p>接下来则需要根据定期存款意愿将客户分组，并进一步进行分析：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//聚合函数分析</span>
<span class="hljs-comment">//根据定期存款意愿将客户分组，并统计各组客户的客户总数，此次访谈的电话联系的平均次数，最后一次电话联系的平均持续时间，早前访谈电话联系的平均次数</span>
dsSubset
.groupBy(<span class="hljs-string">"deposit"</span>)
.agg(count(<span class="hljs-string">"age"</span>).name(<span class="hljs-string">"Total customers"</span>),
round(avg(<span class="hljs-string">"campaign"</span>),<span class="hljs-number">2</span>).name(<span class="hljs-string">"Avgcalls(curr)"</span>),
round(avg(<span class="hljs-string">"dur"</span>),<span class="hljs-number">2</span>).name(<span class="hljs-string">"Avg dur"</span>),	round(avg(<span class="hljs-string">"prev"</span>),<span class="hljs-number">2</span>).name(<span class="hljs-string">"AvgCalls(prev)"</span>)).withColumnRenamed(<span class="hljs-string">"value"</span>,<span class="hljs-string">"TDSubscribed?"</span>)
.show()
</code></pre>
<p>结果为：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/CA/CgqCHl7fP3uAAtbMAAB1XvA5LG4689.png" alt="Drawing 6.png"></p>
<p>这段代码根据年龄将客户分组，并进一步进行分析：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-comment">//根据年龄将客户分组，并统计各组客户的客户总数，此次访谈的电话联系的平均次数，最后一次电话联系的平均持续时间，早前访谈电话联系的平均次数</span>
dsSubset
.groupBy(<span class="hljs-string">"age"</span>)
.agg(count(<span class="hljs-string">"age"</span>).name(<span class="hljs-string">"Total customers"</span>),
round(avg(<span class="hljs-string">"campaign"</span>),<span class="hljs-number">2</span>).name(<span class="hljs-string">"Avgcalls(curr)"</span>),
round(avg(<span class="hljs-string">"dur"</span>),<span class="hljs-number">2</span>).name(<span class="hljs-string">"Avg dur"</span>),
round(avg(<span class="hljs-string">"prev"</span>),<span class="hljs-number">2</span>).name(<span class="hljs-string">"AvgCalls(prev)"</span>)).orderBy(<span class="hljs-string">"age"</span>)
.show()
</code></pre>
<p>结果为：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/BE/Ciqc1F7fP5KAJxseAAGtpeiBSns139.png" alt="Drawing 7.png"></p>
<h3>小结</h3>
<p>数据分析师通过上面这个过程，就可以对数据大致的质量、分布、基本统计信息有了一个基本的印象，这样探索的目的也就达到了。在很多情况下，一些分析师在数据量不大的情况下也喜欢用 Spark 来分析数据，这得益于 DataFrame API 与 Datasets API 的简洁与功能完善。这份数据中，有趣的地方还有很多，你不妨用 Python DataFrame API 与 Spark SQL 尽情探索吧。</p>

---

### 精选评论

##### **森：
> 文中分析师说数据量不大的使用spark，那么数据量大的时候呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里的意思是由于Spark语法比较友好，所以小数据量下就连分析师也爱用Spark，大数据量当然更合适了

