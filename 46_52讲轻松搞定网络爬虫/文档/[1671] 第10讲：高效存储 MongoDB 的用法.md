<p>上节课我们学习了如何用 pyquery 提取 HTML 中的信息，但是当我们成功提取了数据之后，该往哪里存放呢？</p>
<p>用文本文件当然是可以的，但文本存储不方便检索。有没有既方便存，又方便检索的存储方式呢？</p>
<p>当然有，本课时我将为你介绍一个文档型数据库 —— MongoDB。</p>
<p>MongoDB 是由 C++ 语言编写的非关系型数据库，是一个基于分布式文件存储的开源数据库系统，其内容存储形式类似 JSON 对象，它的字段值可以包含其他文档、数组及文档数组，非常灵活。</p>
<p>在这个课时中，我们就来看看 Python 3 下 MongoDB 的存储操作。</p>
<h3>准备工作</h3>
<p>在开始之前，请确保你已经安装好了 MongoDB 并启动了其服务，同时安装好了 Python 的 PyMongo 库。</p>
<p>MongoDB 的安装方式可以参考：<a href="https://cuiqingcai.com/5205.html">https://cuiqingcai.com/5205.html</a>，安装好之后，我们需要把 MongoDB 服务启动起来。</p>
<blockquote>
<p>注意：这里我们为了学习，仅使用 MongoDB 最基本的单机版，MongoDB 还有主从复制、副本集、分片集群等集群架构，可用性可靠性更好，如有需要可以自行搭建相应的集群进行使用。</p>
</blockquote>
<p>启动完成之后，它会默认在本地 localhost 的 27017 端口上运行。</p>
<p>接下来我们需要安装 PyMongo 这个库，它是 Python 用来操作 MongoDB 的第三方库，直接用 pip3 安装即可：<code data-backticks="1">pip3 install pymongo</code>。</p>
<p>更详细的安装方式可以参考：<a href="https://cuiqingcai.com/5230.html">https://cuiqingcai.com/5230.html</a>。</p>
<p>安装完成之后，我们就可以使用 PyMongo 来将数据存储到 MongoDB 了。</p>
<h3>连接 MongoDB</h3>
<p>连接 MongoDB 时，我们需要使用 PyMongo 库里面的 MongoClient。一般来说，我们只需要向其传入 MongoDB 的 IP 及端口即可，其中第一个参数为地址 host，第二个参数为端口 port（如果不给它传递参数，则默认是 27017）：</p>
<pre><code data-language="python" class="lang-python"><span class="hljs-keyword">import</span> pymongo
client = pymongo.MongoClient(host=<span class="hljs-string">'localhost'</span>, port=<span class="hljs-number">27017</span>)
</code></pre>
<p>这样我们就可以创建 MongoDB 的连接对象了。</p>
<p>另外，MongoClient 的第一个参数 host 还可以直接传入 MongoDB 的连接字符串，它以 mongodb 开头，例如：</p>
<pre><code data-language="python" class="lang-python">client = MongoClient(<span class="hljs-string">'mongodb://localhost:27017/'</span>)
</code></pre>
<p>这样也可以达到同样的连接效果。</p>
<h3>指定数据库</h3>
<p>MongoDB 中可以建立多个数据库，接下来我们需要指定操作其中一个数据库。这里我们以 test 数据库作为下一步需要在程序中指定使用的例子：</p>
<pre><code data-language="python" class="lang-python">db = client.test
</code></pre>
<p>这里调用 client 的 test 属性即可返回 test 数据库。当然，我们也可以这样指定：</p>
<pre><code data-language="pyton" class="lang-pyton">db = client['test']
</code></pre>
<p>这两种方式是等价的。</p>
<h3>指定集合</h3>
<p>MongoDB 的每个数据库又包含许多集合（collection），它们类似于关系型数据库中的表。</p>
<p>下一步需要指定要操作的集合，这里我们指定一个名称为 students 的集合。与指定数据库类似，指定集合也有两种方式：</p>
<pre><code data-language="python" class="lang-python">collection = db.students
</code></pre>
<p>或是</p>
<pre><code data-language="python" class="lang-python">collection = db[<span class="hljs-string">'students'</span>]
</code></pre>
<p>这样我们便声明了一个 Collection 对象。</p>
<h3>插入数据</h3>
<p>接下来，便可以插入数据了。我们对 students 这个集合新建一条学生数据，这条数据以字典形式表示：</p>
<pre><code data-language="python" class="lang-python">student = {
    <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170101'</span>,
    <span class="hljs-string">'name'</span>: <span class="hljs-string">'Jordan'</span>,
    <span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>,
    <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>
}
</code></pre>
<p>新建的这条数据里指定了学生的学号、姓名、年龄和性别。接下来，我们直接调用 collection 的 insert 方法即可插入数据，代码如下：</p>
<pre><code data-language="python" class="lang-python">result = collection.insert(student)
print(result)
</code></pre>
<p>在 MongoDB 中，每条数据其实都有一个 _id 属性来唯一标识。如果没有显式指明该属性，MongoDB 会自动产生一个 ObjectId 类型的 _id 属性。insert() 方法会在执行后返回_id 值。</p>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python"><span class="hljs-number">5932</span>a68615c2606814c91f3d
</code></pre>
<p>当然，我们也可以同时插入多条数据，只需要以列表形式传递即可，示例如下：</p>
<pre><code data-language="python" class="lang-python">student1 = {
    <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170101'</span>,
    <span class="hljs-string">'name'</span>: <span class="hljs-string">'Jordan'</span>,
    <span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>,
    <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>
}

student2 = {
    <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170202'</span>,
    <span class="hljs-string">'name'</span>: <span class="hljs-string">'Mike'</span>,
    <span class="hljs-string">'age'</span>: <span class="hljs-number">21</span>,
    <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>
}

result = collection.insert([student1, student2])
print(result)
</code></pre>
<p>返回结果是对应的_id 的集合：</p>
<pre><code data-language="python" class="lang-python">[ObjectId(<span class="hljs-string">'5932a80115c2606a59e8a048'</span>), ObjectId(<span class="hljs-string">'5932a80115c2606a59e8a049'</span>)]
</code></pre>
<p>实际上，在 PyMongo 中，官方已经不推荐使用 insert 方法了。但是如果你要继续使用也没有什么问题。目前，官方推荐使用 insert_one 和 insert_many 方法来分别插入单条记录和多条记录，示例如下：</p>
<pre><code data-language="python" class="lang-python">student = {
    <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170101'</span>,
    <span class="hljs-string">'name'</span>: <span class="hljs-string">'Jordan'</span>,
    <span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>,
    <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>
}

result = collection.insert_one(student)
print(result)
print(result.inserted_id)
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">&lt;pymongo.results.InsertOneResult object at <span class="hljs-number">0x10d68b558</span>&gt;
<span class="hljs-number">5932</span>ab0f15c2606f0c1cf6c5
</code></pre>
<p>与 insert 方法不同，这次返回的是 InsertOneResult 对象，我们可以调用其 inserted_id 属性获取_id。</p>
<p>对于 insert_many 方法，我们可以将数据以列表形式传递，示例如下：</p>
<pre><code data-language="python" class="lang-python">student1 = {
    <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170101'</span>,
    <span class="hljs-string">'name'</span>: <span class="hljs-string">'Jordan'</span>,
    <span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>,
    <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>
}

student2 = {
    <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170202'</span>,
    <span class="hljs-string">'name'</span>: <span class="hljs-string">'Mike'</span>,
    <span class="hljs-string">'age'</span>: <span class="hljs-number">21</span>,
    <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>
}

result = collection.insert_many([student1, student2])
print(result)
print(result.inserted_ids)
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">&lt;pymongo.results.InsertManyResult object at <span class="hljs-number">0x101dea558</span>&gt;
[ObjectId(<span class="hljs-string">'5932abf415c2607083d3b2ac'</span>), ObjectId(<span class="hljs-string">'5932abf415c2607083d3b2ad'</span>)]
</code></pre>
<p>该方法返回的类型是 InsertManyResult，调用 inserted_ids 属性可以获取插入数据的 _id 列表。</p>
<h3>查询</h3>
<p>插入数据后，我们可以利用 find_one 或 find 方法进行查询，其中 find_one 查询得到的是单个结果，find 则返回一个生成器对象。示例如下：</p>
<pre><code data-language="python" class="lang-python">result = collection.find_one({<span class="hljs-string">'name'</span>: <span class="hljs-string">'Mike'</span>})
print(type(result))
print(result)
</code></pre>
<p>这里我们查询 name 为 Mike 的数据，它的返回结果是字典类型，运行结果如下：</p>
<pre><code data-language="python" class="lang-python">&lt;class 'dict'&gt;
{'_id': ObjectId('5932a80115c2606a59e8a049'), 'id': '20170202', 'name': 'Mike', 'age': 21, 'gender': 'male'}
</code></pre>
<p>可以发现，它多了 _id 属性，这就是 MongoDB 在插入过程中自动添加的。</p>
<p>此外，我们也可以根据 ObjectId 来查询，此时需要调用 bson 库里面的 objectid：</p>
<pre><code data-language="python" class="lang-python"><span class="hljs-keyword">from</span> bson.objectid <span class="hljs-keyword">import</span> ObjectId

result = collection.find_one({<span class="hljs-string">'_id'</span>: ObjectId(<span class="hljs-string">'593278c115c2602667ec6bae'</span>)})
print(result)
</code></pre>
<p>其查询结果依然是字典类型，具体如下：</p>
<pre><code data-language="python" class="lang-python">{<span class="hljs-string">'_id'</span>: ObjectId(<span class="hljs-string">'593278c115c2602667ec6bae'</span>), <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170101'</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">'Jordan'</span>, <span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>, <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>}
</code></pre>
<p>如果查询结果不存在，则会返回 None。</p>
<p>对于多条数据的查询，我们可以使用 find 方法。例如，这里查找年龄为 20 的数据，示例如下：</p>
<pre><code data-language="python" class="lang-python">results = collection.find({<span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>})
print(results)
<span class="hljs-keyword">for</span> result <span class="hljs-keyword">in</span> results:
    print(result)
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">&lt;pymongo.cursor.Cursor object at <span class="hljs-number">0x1032d5128</span>&gt;
{<span class="hljs-string">'_id'</span>: ObjectId(<span class="hljs-string">'593278c115c2602667ec6bae'</span>), <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170101'</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">'Jordan'</span>, <span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>, <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>}
{<span class="hljs-string">'_id'</span>: ObjectId(<span class="hljs-string">'593278c815c2602678bb2b8d'</span>), <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170102'</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">'Kevin'</span>, <span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>, <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>}
{<span class="hljs-string">'_id'</span>: ObjectId(<span class="hljs-string">'593278d815c260269d7645a8'</span>), <span class="hljs-string">'id'</span>: <span class="hljs-string">'20170103'</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">'Harden'</span>, <span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>, <span class="hljs-string">'gender'</span>: <span class="hljs-string">'male'</span>}
</code></pre>
<p>返回结果是 Cursor 类型，它相当于一个生成器，我们需要遍历获取的所有结果，其中每个结果都是字典类型。</p>
<p>如果要查询年龄大于 20 的数据，则写法如下：</p>
<pre><code data-language="python" class="lang-python">results = collection.find({<span class="hljs-string">'age'</span>: {<span class="hljs-string">'$gt'</span>: <span class="hljs-number">20</span>}})
</code></pre>
<p>这里查询的条件键值已经不是单纯的数字了，而是一个字典，其键名为比较符号 $gt，意思是大于，键值为 20。</p>
<p>我将比较符号归纳为下表：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/77/5E/CgpOIF5x81GAEHGaAACf0FUPMUU926.png" alt=""></p>
<p>另外，还可以进行正则匹配查询。例如，查询名字以 M 开头的学生数据，示例如下：</p>
<pre><code data-language="python" class="lang-python">results = collection.find({<span class="hljs-string">'name'</span>: {<span class="hljs-string">'$regex'</span>: <span class="hljs-string">'^M.*'</span>}})
</code></pre>
<p>这里使用 $regex 来指定正则匹配，^M.* 代表以 M 开头的正则表达式。</p>
<p>我将一些功能符号归类为下表：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/77/5E/Cgq2xl5x83iAAn4RAAEsbDKOSTc291.png" alt=""><br>
关于这些操作的更详细用法，可以在 MongoDB 官方文档找到：&nbsp;<a href="https://docs.mongodb.com/manual/reference/operator/query/">https://docs.mongodb.com/manual/reference/operator/query/</a>。</p>
<h3>计数</h3>
<p>要统计查询结果有多少条数据，可以调用 count 方法。我们以统计所有数据条数为例：</p>
<pre><code data-language="python" class="lang-python">count = collection.find().count()
print(count)
</code></pre>
<p>我们还可以统计符合某个条件的数据：</p>
<pre><code data-language="python" class="lang-python">count = collection.find({<span class="hljs-string">'age'</span>: <span class="hljs-number">20</span>}).count()
print(count)
</code></pre>
<p>运行结果是一个数值，即符合条件的数据条数。</p>
<h3>排序</h3>
<p>排序时，我们可以直接调用 sort 方法，并在其中传入排序的字段及升降序标志。示例如下：</p>
<pre><code data-language="python" class="lang-python">results = collection.find().sort(<span class="hljs-string">'name'</span>, pymongo.ASCENDING)
print([result[<span class="hljs-string">'name'</span>] <span class="hljs-keyword">for</span> result <span class="hljs-keyword">in</span> results])
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">[<span class="hljs-string">'Harden'</span>, <span class="hljs-string">'Jordan'</span>, <span class="hljs-string">'Kevin'</span>, <span class="hljs-string">'Mark'</span>, <span class="hljs-string">'Mike'</span>]
</code></pre>
<p>这里我们调用 pymongo.ASCENDING 指定升序。如果要降序排列，可以传入 pymongo.DESCENDING。</p>
<h3>偏移</h3>
<p>在某些情况下，我们可能只需要取某几个元素，这时可以利用 skip 方法偏移几个位置，比如偏移 2，就代表忽略前两个元素，得到第 3 个及以后的元素：</p>
<pre><code data-language="python" class="lang-python">results = collection.find().sort(<span class="hljs-string">'name'</span>, pymongo.ASCENDING).skip(<span class="hljs-number">2</span>)
print([result[<span class="hljs-string">'name'</span>] <span class="hljs-keyword">for</span> result <span class="hljs-keyword">in</span> results])
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">[<span class="hljs-string">'Kevin'</span>, <span class="hljs-string">'Mark'</span>, <span class="hljs-string">'Mike'</span>]
</code></pre>
<p>另外，我们还可以用 limit 方法指定要取的结果个数，示例如下：</p>
<pre><code data-language="python" class="lang-python">results = collection.find().sort(<span class="hljs-string">'name'</span>, pymongo.ASCENDING).skip(<span class="hljs-number">2</span>).limit(<span class="hljs-number">2</span>)
print([result[<span class="hljs-string">'name'</span>] <span class="hljs-keyword">for</span> result <span class="hljs-keyword">in</span> results])
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">[<span class="hljs-string">'Kevin'</span>, <span class="hljs-string">'Mark'</span>]
</code></pre>
<p>如果不使用 limit 方法，原本会返回 3 个结果，加了限制后，就会截取两个结果返回。</p>
<p>值得注意的是，在数据量非常庞大的时候，比如在查询千万、亿级别的数据库时，最好不要使用大的偏移量，因为这样很可能导致内存溢出。此时可以使用类似如下操作来查询：</p>
<pre><code data-language="python" class="lang-python"><span class="hljs-keyword">from</span> bson.objectid <span class="hljs-keyword">import</span> ObjectId
collection.find({<span class="hljs-string">'_id'</span>: {<span class="hljs-string">'$gt'</span>: ObjectId(<span class="hljs-string">'593278c815c2602678bb2b8d'</span>)}})
</code></pre>
<p>这时需要记录好上次查询的 _id。</p>
<h3>更新</h3>
<p>对于数据更新，我们可以使用 update 方法，指定更新的条件和更新后的数据即可。例如：</p>
<pre><code data-language="python" class="lang-python">condition = {<span class="hljs-string">'name'</span>: <span class="hljs-string">'Kevin'</span>}
student = collection.find_one(condition)
student[<span class="hljs-string">'age'</span>] = <span class="hljs-number">25</span>
result = collection.update(condition, student)
print(result)
</code></pre>
<p>这里我们要更新 name 为 Kevin 的数据的年龄：首先指定查询条件，然后将数据查询出来，修改年龄后调用 update 方法将原条件和修改后的数据传入。</p>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">{<span class="hljs-string">'ok'</span>: <span class="hljs-number">1</span>, <span class="hljs-string">'nModified'</span>: <span class="hljs-number">1</span>, <span class="hljs-string">'n'</span>: <span class="hljs-number">1</span>, <span class="hljs-string">'updatedExisting'</span>: <span class="hljs-literal">True</span>}
</code></pre>
<p>返回结果是字典形式，ok 代表执行成功，nModified 代表影响的数据条数。</p>
<p>另外，我们也可以使用 $set 操作符对数据进行更新，代码如下：</p>
<pre><code data-language="python" class="lang-python">result = collection.update(condition, {<span class="hljs-string">'$set'</span>: student})
</code></pre>
<p>这样可以只更新 student 字典内存在的字段。如果原先还有其他字段，则不会更新，也不会删除。而如果不用 $set 的话，则会把之前的数据全部用 student 字典替换；如果原本存在其他字段，则会被删除。</p>
<p>另外，update 方法其实也是官方不推荐使用的方法。这里也分为 update_one 方法和 update_many 方法，用法更加严格，它们的第 2 个参数需要使用 $ 类型操作符作为字典的键名，示例如下：</p>
<pre><code data-language="python" class="lang-python">condition = {<span class="hljs-string">'name'</span>: <span class="hljs-string">'Kevin'</span>}
student = collection.find_one(condition)
student[<span class="hljs-string">'age'</span>] = <span class="hljs-number">26</span>
result = collection.update_one(condition, {<span class="hljs-string">'$set'</span>: student})
print(result)
print(result.matched_count, result.modified_count)
</code></pre>
<p>上面的例子中调用了 update_one 方法，使得第 2 个参数不能再直接传入修改后的字典，而是需要使用 {'$set': student} 这样的形式，其返回结果是 UpdateResult 类型。然后分别调用 matched_count 和 modified_count 属性，可以获得匹配的数据条数和影响的数据条数。</p>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">&lt;pymongo.results.UpdateResult object at <span class="hljs-number">0x10d17b678</span>&gt;
<span class="hljs-number">1</span> <span class="hljs-number">0</span>
</code></pre>
<p>我们再看一个例子：</p>
<pre><code data-language="python" class="lang-python">condition = {<span class="hljs-string">'age'</span>: {<span class="hljs-string">'$gt'</span>: <span class="hljs-number">20</span>}}
result = collection.update_one(condition, {<span class="hljs-string">'$inc'</span>: {<span class="hljs-string">'age'</span>: <span class="hljs-number">1</span>}})
print(result)
print(result.matched_count, result.modified_count)
</code></pre>
<p>这里指定查询条件为年龄大于 20，然后更新条件为 {'$inc': {'age': 1}}，表示年龄加 1，执行之后会将第一条符合条件的数据年龄加 1。</p>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">&lt;pymongo.results.UpdateResult object at <span class="hljs-number">0x10b8874c8</span>&gt;
<span class="hljs-number">1</span> <span class="hljs-number">1</span>
</code></pre>
<p>可以看到匹配条数为 1 条，影响条数也为 1 条。</p>
<p>如果调用 update_many 方法，则会将所有符合条件的数据都更新，示例如下：</p>
<pre><code data-language="python" class="lang-python">condition = {<span class="hljs-string">'age'</span>: {<span class="hljs-string">'$gt'</span>: <span class="hljs-number">20</span>}}
result = collection.update_many(condition, {<span class="hljs-string">'$inc'</span>: {<span class="hljs-string">'age'</span>: <span class="hljs-number">1</span>}})
print(result)
print(result.matched_count, result.modified_count)
</code></pre>
<p>这时匹配条数就不再为 1 条了，运行结果如下：</p>
<pre><code data-language="python" class="lang-python">&lt;pymongo.results.UpdateResult object at <span class="hljs-number">0x10c6384c8</span>&gt;
<span class="hljs-number">3</span> <span class="hljs-number">3</span>
</code></pre>
<p>可以看到，这时所有匹配到的数据都会被更新。</p>
<h3>删除</h3>
<p>删除操作比较简单，直接调用 remove 方法指定删除的条件即可，此时符合条件的所有数据均会被删除。</p>
<p>示例如下：</p>
<pre><code data-language="python" class="lang-python">result = collection.remove({<span class="hljs-string">'name'</span>: <span class="hljs-string">'Kevin'</span>})
print(result)
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">{<span class="hljs-string">'ok'</span>: <span class="hljs-number">1</span>, <span class="hljs-string">'n'</span>: <span class="hljs-number">1</span>}
</code></pre>
<p>另外，这里依然存在两个新的推荐方法 —— delete_one 和 delete_many，示例如下：</p>
<pre><code data-language="python" class="lang-python">result = collection.delete_one({<span class="hljs-string">'name'</span>: <span class="hljs-string">'Kevin'</span>})
print(result)
print(result.deleted_count)
result = collection.delete_many({<span class="hljs-string">'age'</span>: {<span class="hljs-string">'$lt'</span>: <span class="hljs-number">25</span>}})
print(result.deleted_count)
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="python" class="lang-python">&lt;pymongo.results.DeleteResult object at <span class="hljs-number">0x10e6ba4c8</span>&gt;
<span class="hljs-number">1</span>
<span class="hljs-number">4</span>
</code></pre>
<p>delete_one 即删除第一条符合条件的数据，delete_many 即删除所有符合条件的数据。它们的返回结果都是 DeleteResult 类型，可以调用 deleted_count 属性获取删除的数据条数。</p>
<h3>其他操作</h3>
<p>另外，PyMongo 还提供了一些组合方法，如 find_one_and_delete、find_one_and_replace 和 find_one_and_update，它们分别用于查找后删除、替换和更新操作，其使用方法与上述方法基本一致。</p>
<p>另外，我们还可以对索引进行操作，相关方法有 create_index、create_indexes 和 drop_index 等。</p>
<p>关于 PyMongo 的详细用法，可以参见官方文档：<a href="http://api.mongodb.com/python/current/api/pymongo/collection.html">http://api.mongodb.com/python/current/api/pymongo/collection.html</a>。</p>
<p>另外，还有对数据库和集合本身等的一些操作，这里不再一一讲解，可以参见官方文档：<a href="http://api.mongodb.com/python/current/api/pymongo/">http://api.mongodb.com/python/current/api/pymongo/</a>。</p>
<p>本课时的内容我们就讲到这里了，你是不是对如何使用 PyMongo 操作 MongoDB 进行数据增删改查更加熟悉了呢？下一课时我将会带你在实战案例中应用这些操作进行数据存储。</p>

---

### 精选评论

##### **8093：
> 安装bson库后，<div>from bson.objectid import ObjectId<br></div><div>报错：ImportError: cannot import name 'abc' from 'bson.py3compat'</div><div>解决方案：</div><div>pip uninstall bson</div><div>pip uninstall pymongo</div><div>pip install bson</div><div>pip install pymongo</div><div><br></div>

##### **3202：
> 崔老师讲得很棒！小白的我都还能听懂！技巧重点都讲到了！希望以后也能像老师一样成为技术大牛！

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 飞驰吧，少年！朝着自己的目标努力吧！

##### **9799：
> 不能用mysql吗？一定要用mongodb?&nbsp;

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; mysql 对于一些结构化或嵌套类型的数据存储不太方便，而且需要额外维护字段信息，相对麻烦，这里选取了更方便的 MongoDB。MongoDB 的性能也很强，也适合存储这类数据。

##### **鹏：
> 收获很大，本章节先介绍mongo，然依次叙述mongo用法，连接，增，删，改，查（与章节顺序不同）mongo命令使用

##### **0451：
> 平时用用爬虫抓取数据，老师推荐用mangodb还是redis？哪个学习起来更加容易上手呢？

##### *彤：
> 老师讲得非常精彩，数据库小白听起来很轻松

##### **0552：
> 老师，我mongodb服务启动不起来，说什么error receiveing request from client 怎么办

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个检查下 MongoDB 是否安装成功吧，比如数据文件夹、配置文件字段是否有问题。4.0 应该安装相对简单一些。再不行的话再搜搜重装下吧。

