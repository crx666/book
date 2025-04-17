# ElasticSearch

Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎，存储的数据为json格式。

1. 索引（index）

   相当于mysql的数据表

   ```
   PUT /dog		//创建索引
   DELETE /dog		//删除索引
   GET /dog		//查询索引
   HEAD /dog		//是否存在索引
   ```

2. 文档（document）

   相当于mysql表中的一条数据

   ```
   PUT /dog/_doc/1
   {
     "name":"dog1",
     "age":1
   }					//创建索引并且增加一条数据

   POST /dog/_doc/1/_update	//更新数据
   {
      "name":"dog111",
      "age":1
   }

   DELETE /dog/_doc/1?timeout=5m	//删除指定数据，超时时间为5分钟
   POST /dog/_doc/_delete_by_query	//删除所有数据
   {
   	"query":{
         "match_all":{}
   	} 
   }

   GET /dog/_doc/1		//查找指定数据
   GET /dog/_doc/_search	//查找所有数据
   GET /dog/_doc/_search	//查找所有名字包含dog的数据
   {
     "query":{
       "match":{
         "name":"dog"	
       }
     }
   }

   GET /dog/_doc/_search	
   {
     "query":{
       "match":{
         "name":"dog"	
       }
     },
     "size":10, 	//跳过0条数据
     "from":0,		//指定返回10条
     "sort":{
       "age":"desc"	//年龄倒序排序
     },
     "_source":["name"], //只返回name字段
     ........
   }

   GET /dog/_doc/_search		//查找名字包含dog和cat的数据
   {
     "query": {
       "bool": {
         "must": [
           { "match": { "name": "dog" } },
           { "match": { "name": "cat" } }
         ]
       }
     }
   }

   GET /dog/_doc/_search		//查找名字包含dog或cat的数据
   {
     "query" : { 
       "match" : {
         "name" : "dog cat"
         }
       }
   }
   ```

3. mapping

   相当于mysql的表结构

   ```
   PUT /dog/_doc/1
   {
     "mappings": {
       "properties": {
         "name": { "type": "keyword" }	//name字段
       }
     }
   }
   ```

   ​



#  倒排索引

倒排索引就是也叫反向索引，就是根据value来查询key，正向索引是根据key来查询value。

倒排索引分为三个部分：

![img](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127172829635-1286260863.png)

**Term（单词）**：一段文本经过分析器分析以后就会输出一串单词，这一个一个的就叫做Term（直译为：单词）

**Term Dictionary（单词字典）**：顾名思义，它里面维护的是Term，可以理解为Term的集合

**Term Index（单词索引）**：为了更快的找到某个单词，我们为单词建立索引

**Posting List（倒排列表）**：倒排列表记录了出现过某个单词的所有文档的文档列表及单词在该文档中出现的位置信息，每条记录称为一个倒排项(Posting)。根据倒排列表，即可获知哪些文档包含某个单词。（PS：实际的倒排列表中并不只是存了文档ID这么简单，还有一些其它的信息，比如：词频（Term出现的次数）、偏移量（offset）等，可以想象成是Python中的元组，或者Java中的对象）

（PS：如果类比现代汉语词典的话，那么Term就相当于词语，Term Dictionary相当于汉语词典本身，Term Index相当于词典的目录索引）



倒排索引的原理：term index以树的形式缓存在内存中，查询时通过value首先从term index中查到对应的term dictionary的block位置之后，再去查找Posting List，Posting List是一个数组，存储了所有符合某个Term的文档ID，通过文档id再去查找对应的trem。





首先有这么一组数据，它有四个字段：分别是name，gender，age，address。

![img](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127173241683-1331385372.png)

下图代表各个字段的倒排索引：

**name字段：**

![img](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127175423615-230290274.png)

**age字段：**

![img](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127175627644-1013476663.png)

**gender字段：**

![img](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127175809626-1224287371.png)

**address字段：**

![img](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127180053644-1305820142.png)



Elasticsearch分别为每个字段都建立了一个倒排索引。比如，在上面“张三”、“北京市”、22 这些都是Term，而[1，3]就是Posting List。Posting list就是一个数组，存储了所有符合某个Term id



![img](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127184959667-1135956344.png)