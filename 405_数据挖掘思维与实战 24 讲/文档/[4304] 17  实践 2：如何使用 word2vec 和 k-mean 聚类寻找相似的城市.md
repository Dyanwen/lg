<p data-nodeid="16590" class="">在第一个实践课（使用 XGB 实现酒店信息消歧）中其实没有涉及太多的代码，主要是以介绍思路为主。在这一课时中，我将提供一个较为完整的代码，带领你亲自实践一下。</p>


<h3 data-nodeid="16250">理解业务</h3>
<p data-nodeid="16251">在旅行场景下，城市——我们通常称为目的地，是一个很重要的信息。根据用户对于目的地的偏好，我们既可以把目的地作为一个特征用于推荐系统中，也可以把目的地当作一个被推荐的信息直接推荐给用户。所以，我们有一个需求，就是把相似的目的地整理出来，然后可以通过这些相似目的地做相关推荐，或者是相关目的地的推荐。</p>
<h3 data-nodeid="16252">理解数据</h3>
<p data-nodeid="16253">可以想到，这是一个比较典型的聚类问题，我们只要能够把相似的城市按照一定的相关性聚在一起，就可以完成我们的需求，当然具体的效果要根据结果不断地进行调整。</p>
<p data-nodeid="16254">那我们就来看一下我们的数据。</p>
<p data-nodeid="17032">思来想去我们只有很多目的地的名字，但这些目的地并没有什么统一标准的特征可以给我们做向量，那么该怎么去给这些目的地计算相关性呢？</p>
<p data-nodeid="17033" class=""><img src="https://s0.lgstatic.com/i/image/M00/54/D7/Ciqc1F9pooWAPjf7AALE-P3miZc088.png" alt="Drawing 0.png" data-nodeid="17037"></p>


<p data-nodeid="16257">这时不禁想到，我们有很多用户写过游记，这些游记里总会出现各种各样的目的地名字，对于相似的目的地，那用户所写的内容也会有一定的相似性，不管是地理位置接近，还是消费价位类似，或者是可以玩的内容存在一定的相似性。</p>
<p data-nodeid="16258">总之，我们可以靠这些内容把这些城市的名字关联起来，而且不同于结构化的信息，游记是用户自己来写的内容，里面对于目的地的认知也是用户的认知，所以如果我们能够从中发现关联性，再应用到用户身上也是比较合理的。比如说“三亚”如果只是按客观属性来划分，那应该是“海边”，但是很多用户去三亚，除了看海本身，还有家庭出游等，这些是只能从用户的角度才会产生的认知。</p>
<p data-nodeid="16259">这里，我们就要用到一个 Word2Vec 算法，它可以学习输入的文本，并输出一个词向量模型，经过 Word2Vec 算法处理之后，每一个词都会变成一个预设长度的数值向量。这个算法会在后面的章节进行更详细的讲解，这里我们大概知道它的功能就可以了。下面我们进入到具体代码实现的环节，看看如何训练一个这样的模型。</p>
<h3 data-nodeid="16260">准备数据与模型训练</h3>
<h4 data-nodeid="16261">准备数据</h4>
<p data-nodeid="16262">我们获取所有需要用到的文本数据，在这里使用了全量的游记文本数据。<strong data-nodeid="16310">我们首先要对数据进行清洗，去除掉异常的数据</strong>，比如内容过短、获取失败，或者是存在特殊字符、使用纯英文 / 泰语写的游记，等等。</p>
<p data-nodeid="16263"><strong data-nodeid="16319">完成了这个步骤之后我们要对文本内容进行分词</strong>，因为我期望 Word2Vec 最终构建的向量是词级别的。<strong data-nodeid="16320">完成分词之后，我们把数据存储在文本文件中</strong>，其中每一行是一篇内容。</p>
<p data-nodeid="16264">接下来就要训练我们的 Word2Vec模型了。</p>
<h4 data-nodeid="16265">训练 Word2Vec 模型</h4>
<p data-nodeid="16266"><strong data-nodeid="16327">这里我们使用了一个新的算法包：Gensim</strong>。不知道你是否还记得我在之前介绍过这个工具包，它主要用于从原始的非结构化文本信息中，通过无监督算法学习文本向量表达。这里面支持 TF-IDF、LSA、LDA 和 Word2Vec 等多种算法模型。来看一下代码。</p>
<pre class="lang-python" data-nodeid="16267"><code data-language="python"><span class="hljs-keyword">import</span> gensim <span class="hljs-comment">#引入gensim</span>
<span class="hljs-keyword">import</span> os
<span class="hljs-keyword">import</span> re
<span class="hljs-keyword">import</span> sys
<span class="hljs-keyword">import</span> multiprocessing <span class="hljs-comment">#引入多线程操作</span>
<span class="hljs-keyword">from</span> time <span class="hljs-keyword">import</span> time
<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">getSentence</span>(<span class="hljs-params">object</span>):</span>
<span class="hljs-comment">#初始化，获取文件路径</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-params">self, dirname</span>):</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;self.dirname = dirname
</code></pre>
<p data-nodeid="16268">文本可以存储在多个文本文件中，存放在一个文件目录下，这里构建了一个迭代方法，循环读取目录下的所有文件。</p>
<p data-nodeid="17478">我这里使用的文件目录为 traindata，在 traindata 下面有 31 个语料文件，其中每个有 1G 左右，如下图所示。</p>
<p data-nodeid="17479" class=""><img src="https://s0.lgstatic.com/i/image/M00/54/E3/CgqCHl9popKALVmjAAA0K1jp_z4167.png" alt="Drawing 1.png" data-nodeid="17483"></p>


<pre class="lang-python" data-nodeid="16271"><code data-language="python"><span class="hljs-comment">#构建一个迭代器</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__iter__</span>(<span class="hljs-params">self</span>):</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span> root, dirs, files <span class="hljs-keyword">in</span> os.walk(self.dirname):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span> filename <span class="hljs-keyword">in</span> files:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;file_path = root + <span class="hljs-string">'/'</span> + filename
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span> line <span class="hljs-keyword">in</span> open(file_path):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>:
&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp; <span class="hljs-comment">#清除异常数据，主要是去除空白符以及长度为0的内容</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s_line = line.strip()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span> s_line== <span class="hljs-string">""</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">continue</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">#把句子拆成词</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;word_line = [word <span class="hljs-keyword">for</span> word <span class="hljs-keyword">in</span> s_line.split( )]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">yield</span> word_line
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">except</span> Exception:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">"catch exception"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">yield</span> <span class="hljs-string">""</span>
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">'__main__'</span>:
<span class="hljs-comment">#记录一个起始时间</span>
&nbsp;&nbsp;&nbsp;&nbsp;begin = time()
<span class="hljs-comment">#获取句子迭代器</span>
&nbsp;&nbsp;&nbsp;&nbsp;sentences = getSentence(<span class="hljs-string">"traindata"</span>)
<span class="hljs-comment">#训练word2vec模型&nbsp;使用句子迭代器作为语料的输入，设定的最终向量长度为200维；窗口长度为15；词的最小计数为10，词频少于10的词不会进行计算；使用并行处理</span>
&nbsp;&nbsp;&nbsp;&nbsp;model = gensim.models.Word2Vec(sentences,size=<span class="hljs-number">200</span>,window=<span class="hljs-number">15</span>,min_count=<span class="hljs-number">10</span>, workers=multiprocessing.cpu_count())
<span class="hljs-comment">#模型存储，这块记得先预先新建一个model路径，或者也可以增加一段代码来识别是否已经创建，如果没有则新建一个路径</span>
&nbsp;&nbsp;&nbsp;&nbsp;model.save(<span class="hljs-string">"model/word2vec_gensim"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;model.wv.save_word2vec_format(<span class="hljs-string">"model/word2vec_org"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"model/vocabulary"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;binary=<span class="hljs-literal">False</span>)
&nbsp;&nbsp;&nbsp;&nbsp;end = time()
<span class="hljs-comment">#输出运算所用时间</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">print</span> (<span class="hljs-string">"Total procesing time: %d seconds"</span> % (end - begin))
</code></pre>
<p data-nodeid="16272">在正常的情况下，我们会在 model 路径下看到几个文件。其中比较重要的两个，一个 vocabulary 是词典文件，记录了出现过的词汇以及词汇出现的次数；一个 word2vec_gensim 是生成的向量文件。</p>
<p data-nodeid="16273">通过上面的方法，我们成功获取到了很多词汇的向量，这里我的词汇量大概有 1000w 左右。但是我们这次所需要的是寻找相似城市，所以对于那些非城市名字的词汇就没有什么价值了。</p>
<p data-nodeid="16274">于是我们这里使用我们自己的城市词库与词汇表进行匹配，对于没有在词汇表中出现过的城市名称也没有办法计算，要把这部分剔除掉。不用担心，如果这么多的语料都没有出现过的城市也一定是没有人去过的城市。</p>
<h4 data-nodeid="16275">训练 K-means 模型</h4>
<p data-nodeid="16276">下面我们就可以开始训练我们的 K-means 模型了。像我们前面用过的一样，K-means 是在 sklearn 里面的一个模块。具体步骤如下所示。</p>
<pre class="lang-python" data-nodeid="16277"><code data-language="python"><span class="hljs-keyword">import</span> gensim
<span class="hljs-keyword">from</span> sklearn.cluster <span class="hljs-keyword">import</span> KMeans
<span class="hljs-keyword">from</span> sklearn.externals <span class="hljs-keyword">import</span> joblib
<span class="hljs-keyword">from</span> time <span class="hljs-keyword">import</span> time
<span class="hljs-comment">#加载之前已经训练好的word2vec模型</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">load_model</span>():</span>
&nbsp;&nbsp;&nbsp;&nbsp;model = gensim.models.Word2Vec.load(<span class="hljs-string">'../word2vec/model/word2vec_gensim'</span>)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> model
<span class="hljs-comment">#加载城市名称词库</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">load_filterword</span>():</span>
&nbsp;&nbsp;&nbsp;&nbsp;fd = open(<span class="hljs-string">"mddwords.txt"</span>,<span class="hljs-string">"r"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;filterword=[]
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span> line <span class="hljs-keyword">in</span> fd.readlines():
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;line=line.strip()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;filterword.append(line)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> filterword
<span class="hljs-keyword">if</span> __name__==<span class="hljs-string">"__main__"</span>:
&nbsp;&nbsp;&nbsp;&nbsp;start = time()
<span class="hljs-comment">#加载word2vec模型</span>
&nbsp;&nbsp;&nbsp;&nbsp;model = load_model()
<span class="hljs-comment">#加载词汇表</span>
&nbsp;&nbsp;&nbsp;&nbsp;filterword = load_filterword()
<span class="hljs-comment">#输出词汇表长度</span>
&nbsp;&nbsp;&nbsp;&nbsp;print(len(filterword))
&nbsp;&nbsp;&nbsp;&nbsp;wordvector = []
&nbsp;&nbsp;&nbsp;&nbsp;filterkey={}
<span class="hljs-comment">#获取我们的城市名称词库的词向量</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span> word <span class="hljs-keyword">in</span> filterword:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;wordvector.append(model[word])
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;filterkey[word]=model[word]
&nbsp; &nbsp;<span class="hljs-comment">#输出词汇数量</span>
&nbsp;&nbsp;&nbsp;&nbsp;print(len(wordvector))
<span class="hljs-comment">#训练K-means模型，这里代码设置的聚类数为2000，最大迭代次数为100，n_jobs设置的是有多少个任务同时在跑，这样可以进行多组实验来消除初始化点带来的影响</span>
&nbsp;&nbsp;&nbsp;&nbsp;clf = KMeans(n_clusters=<span class="hljs-number">2000</span>,max_iter=<span class="hljs-number">100</span>,n_jobs=<span class="hljs-number">10</span>)
&nbsp;&nbsp;&nbsp;&nbsp;s = clf.fit_predict(wordvector)
<span class="hljs-comment">#把模型保存下来</span>
&nbsp;&nbsp;&nbsp;&nbsp;joblib.dump(clf,<span class="hljs-string">"kmeans_mdd2000.pkl"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;labels = clf.labels_
&nbsp;&nbsp;&nbsp;&nbsp;labellist = labels.tolist()
&nbsp;&nbsp;&nbsp;&nbsp;print(clf.inertia_)
<span class="hljs-comment">#把所有城市名称的聚类标签保存下来</span>
&nbsp;&nbsp;&nbsp;&nbsp;fp = open(<span class="hljs-string">"label_mdd2000"</span>,<span class="hljs-string">'w'</span>)
&nbsp;&nbsp;&nbsp;&nbsp;fp.write(str(labellist))
&nbsp;&nbsp;&nbsp;&nbsp;fp.close()
<span class="hljs-comment">#把所有城市名称保存下来，其中顺序与聚类标签顺序一致</span>
&nbsp;&nbsp;&nbsp;&nbsp;fp1 = open(<span class="hljs-string">"keys_mdd2000"</span>,<span class="hljs-string">'w'</span>)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span> key <span class="hljs-keyword">in</span> filterkey:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fp1.write(key+<span class="hljs-string">'\n'</span>)
&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">"over"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;end = time()
&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">"use time"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;print(end-start)
</code></pre>
<p data-nodeid="16278">经过上面的步骤，我们就训练好了 K-means 模型，当然，经过反复尝试，最终确定的不是 2000 这个簇数量，而是使用了 100 个簇的结果。我们尝试了 50、100、200、500、1000、2000 等多个聚类的结果，经过我们最后的对比评估，100 个簇的时候效果较好，于是我们最终选择了这个模型。</p>
<p data-nodeid="17924">下图是我从结果中抽了一些簇的 TOP 结果生成的图片，可以看到聚类的效果还是很不错的。比如右下角那一簇基本都是日本关西的城市名字，左下角基本都是川藏线上的地点。</p>
<p data-nodeid="17925" class=""><img src="https://s0.lgstatic.com/i/image/M00/54/E3/CgqCHl9poqOAYQQKAAlxp_B8UwA621.png" alt="Drawing 2.png" data-nodeid="17929"></p>


<p data-nodeid="16281">有了已经训练好的模型，我们就知道了这些相似城市的名称以及它们所属的簇。接下来我们要做的，就是<strong data-nodeid="16350">把这些数据存储到数据库中，并在具体的业务中进行应用</strong>了。</p>
<p data-nodeid="16282">当然，随着时间的推移，在积累了一段时间的数据之后，我们还要对模型进行重新迭代，以期望获得更好的结果。</p>
<h3 data-nodeid="18154" class="">总结</h3>

<p data-nodeid="16284">在这一节实践课程中，我着重介绍了整个模型训练环节的代码，其中主要写了两段代码，分别训练了 Word2Vec 模型和 K-means 模型。除了数据部分，这些代码几乎可以复制即运行。</p>
<p data-nodeid="18380">到这一课时，关于聚类问题的内容就告一段落了，在数据缺少标注的时候，聚类算法是十分常用的，它可以帮助我们了解数据情况。当然，聚类方法也存在一些局限，还需要在日常的工作中多加练习，不断积累自己的经验。</p>


<blockquote data-nodeid="16287">
<p data-nodeid="16288">点击下方链接查看源代码（不定时更新）以及相关工具：<br>
<a href="https://github.com/icegomic/GomicDatamining/tree/master/LagouCodes" data-nodeid="16361">https://github.com/icegomic/GomicDatamining/tree/master/LagouCodes</a></p>
</blockquote>

---

### 精选评论

##### **灏：
> 当特征维度比较高时候 是不是使用余弦距离去做kmeans更合适呀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 其实都是可以的，可以尝试一下，看看效果

##### **辉：
> 数据在哪里可以获取？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 下方有GitHub的地址哦

##### **3382：
> 老师提纲挈领，讲得很精彩。配上实列，干货满满，受益良多。小白问题，请问n_clusters=2000这个数值是根据经验吗？谢谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 其实经验也是经过不断的尝试获得的，要先了解数据，然后进行估计和多次试验

