# ElasticSearch分析
分析是字符串转换过程，这个过程将字符串转换成可搜索的倒排索引。分析被分析器执行，分析器可以时内建的，也可以是定制的。 

## 索引时分析
例如， 在索引时刻，内建的english 分析器将会转换以下语句：
```"The QUICK brown foxes jumped over the lazy dog!"```
成为不通的关键字。 然后将每个关键字转换成小写，清除停用词（"the") 和化简为词干（foxes->fox,jumped->jump,lazy->lazi）。最后，以下词汇将会被添加到倒排索引中：
```[ quick, brown, fox, jump, over, lazi, dog ]```
### 制定索引时分析器
mapping中的每一个```text```字段都可以指定它自己的分析器：

```
PUT my_index
{  
  "mappings": {  
    "_doc": {  
      "properties": {  
        "title": {  
          "type":     "text",  
          "analyzer": "standard"  
        }  
      }  
    }   
  } 
 }
```
在索引时，如果没有被指定，将会寻找es索引settings中名为```default```的分析器。 如果都没有指定，默认使用```standard```分析器。 

##  查找时分析
在类似于```match```查询的全文检索中，检索时，同样的过程也会应该在查询字符串中，将查询文本转换成那些存储在倒排索引中关键字的格式。
例如，一个用户可能会查找
```"a quick fox"```
这串文本将被```english```分析器转换成以下关键字：
```[ quick, fox ]```
尽管在查询串中抽取到的词，没有出现在原始文档中（quick vs QUICK , fox vs foxes）, 由于我们应用了同样的分析器到原始文档和查询字符串中，查询字符串中的词语实际上匹配了原始文档存储在倒排索引中的词语。
### 指定搜索时分析器
通常索引和搜索都应使用相同的分析器，并且全文查询（如match查询）将使用映射查找该分析器以用于每个字段。
分析器按照以下顺序查找以应用在特殊的字段上： 
* 查询本身指定的分析器
* mapping参数中的```search_analyzer```
* mapping参数中的```analyzer```
* 在索引settings名为```default_search``` 的分析器
* 在索引settings名为```default``` 的分析器
* ```standard```分析器
