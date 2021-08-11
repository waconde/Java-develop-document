# 分析器基建
## 字符过滤器
### HTML去除过滤器（HTML Strip Character Filter） 
HTML去除过滤器会去除<b>之类的HTML元素，并解码`＆amp;`之类的HTML实体。通过向"char_filter"属性中添加`html_strip`元素指定。

示例：

```
POST _analyze
{
  "tokenizer":      "keyword", 
  "char_filter":  [ "html_strip" ],
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}
```
关键词分词器（`keyword`分词器）仅返回一个词项。 上面的示例返回的词项是： 

```
[ \nI'm so happy!\n ]
```
同样的例子，如果使用的分词器是标准分词器（`stadard`分词器），将会返回以下词项： 

```
	[ I'm, so, happy ]
```

#### 配置
HTML去除过滤器接受一下参数： 

`escaped_tags`:一个HTML标签数组，数组中的HTML标签将不会被去除。 

#### 示例配置
在这个例子中，我们配置HTML去除过滤器不需要去除`<b>`标签： 

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "keyword",
          "char_filter": ["my_char_filter"]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "html_strip",
          "escaped_tags": ["b"]
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}
```
上面示例会产生下列词项： 

```
[ \nI'm so <b>happy</b>!\n ]
```

### 字符映射过滤器（Mapping Character Filter）
字符映射过滤器可以将任意指定的字符替换成指定的替换。 字符映射过滤器接受一个键值对映射表，当遇到与键值对映射表中的键相同的字符串时，使用该键所对应的值对字符串替换。匹配是贪婪的，最长模式匹配结果会返回。允许替换成空串。 

#### 配置
字符映射过滤器接受一下参数： 

`mappings`:一个映射数组，每一个元素都是`key => value`的格式
---
`mappings_path`:一个路径，`config`的路径，既可以是绝对路径，也可以是相对路径，指向一个UTF-8编码的映射文件，映射文件中的每一行都是`key => value`的映射。
必须提供`mappings`或`mappings_path`参数。 
#### 示例配置
在这个示例中，我们字符映射过滤器去使用拉丁数字替换阿拉伯数字

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "keyword",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            "٠ => 0",
            "١ => 1",
            "٢ => 2",
            "٣ => 3",
            "٤ => 4",
            "٥ => 5",
            "٦ => 6",
            "٧ => 7",
            "٨ => 8",
            "٩ => 9"
          ]
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "My license plate is ٢٥٠١٥"
}
```
以上示例产生一下词项

```
[ My license plate is 25015 ]
```
键和值可以是多个字符。以下示例将`:)`和`:(`表情替换成等价的文本： 

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            ":) => _happy_",
            ":( => _sad_"
          ]
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "I'm delighted about it :("
}
```
这个示例产生以下词项： 
```
[ I'm, delighted, about, it, _sad_ ]
```


### 模式匹配过滤器（Pattern Replace Character Filter）
模式匹配过滤器将任何匹配表达式的串替换指定的替换串。 模式匹配过滤器使用正则表达式匹配需要替换字符串，并将匹配的结果用相应的替换串替换。替换字符串可以引用正则表达式中的捕获组。

正则表达式使用的是java的正则表达式，使用的时候需要小心表达式的效率问题。 

#### 配置
模式匹配过滤器接受下列参数： 

`pattern`: 一个java的正则表达式，必须给出
---
`replacement`:用于替换的串，可以使用`$1`..`$9`语法引用捕获组
---
`flags`:Java正则表达是的标志。标志之间使用管道符隔开，例如，`"CASE_INSENSITIVE|COMMENTS"`。

#### 示例配置
在这个例子中，我们配置字符过滤器为`pattern_replace`(模式匹配过滤器)，用下划线替换掉在数字中嵌入的破折号： 

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "pattern_replace",
          "pattern": "(\\d+)-(?=\\d)",
          "replacement": "$1_"
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "My credit card is 123-456-789"
}
```
以上例子将会产生以下词项： 

```
[ My, credit, card, is, 123_456_789 ]
```
注意： 使用替换字符过滤器若更改原始文本的长度，虽然可以用于搜索，但会导致不正确的高亮显示，如以下示例所示。

本示例在遇到小写字母后跟大写字母（即`fooBarBaz`→`foo Bar Baz`）时插入一个空格，从而允许单独查询驼峰单词

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "my_char_filter"
          ],
          "filter": [
            "lowercase"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "pattern_replace",
          "pattern": "(?<=\\p{Lower})(?=\\p{Upper})",
          "replacement": " "
        }
      }
    }
  },
  "mappings": {
    "_doc": {
      "properties": {
        "text": {
          "type": "text",
          "analyzer": "my_analyzer"
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "The fooBarBaz method"
}
```
以上示例会返回以下词项： 

```
[ the, foo, bar, baz, method ]
```
查询为`bar` 时返回的文档时正确的，但是高亮的结果是错误的，应为我们的字符过滤器改变了原始字符的长度： 

```
PUT my_index/_doc/1?refresh
{
  "text": "The fooBarBaz method"
}

GET my_index/_search
{
  "query": {
    "match": {
      "text": "bar"
    }
  },
  "highlight": {
    "fields": {
      "text": {}
    }
  }
}
```
上面的输出为：

```
{
  "timed_out": false,
  "took": $body.took,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "my_index",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "text": "The fooBarBaz method"
        },
        "highlight": {
          "text": [
            "The foo<em>Ba</em>rBaz method" 
          ]
        }
      }
    ]
  }
}
```