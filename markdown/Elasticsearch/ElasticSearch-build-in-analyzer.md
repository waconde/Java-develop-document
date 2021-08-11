# ElasticSearch内建分析器
 Elasticsearch附带了很多内建分析器，这些分析器不需要进行配置便可以直接在索引上使用： 
 
## 标准分析器
标准分析器使用unicode文本分割算法(Unicode Text Segmentation algorithm)以词为边界将文本分成词项。 它删除了大多数的标点符号，将词项小写，同时支持删除停用词。 通过配置"analyzer"为`standard`启用

## 简单分析器
简单分析器在遇到非字母字符时会将文本分为多个词项。小写所有词项。通过配置"analyzer"为`simple`启用

## 空白分析器
空白分析器在遇到空白字符时会将文本分为多个词项。小写所有词项。通过配置"analyzer"为`whitespace`启用

## 停用词分析器
停用词分析器有些类似于简单分析器，但是支持删除停用词。通过配置"analyzer"为`stop`启用

## 关键词分析器
关键词分析器不只从任何分析，无论接收到的是什么文本，都会将给定的文本作为一个词项原样返回。 通过配置"analyzer"为`keyword`启用

## 正则分析器
正则分析器通过使用正则表达式将文本分割成词项。支持消息词项和删除停用词。通过配置"analyzer"为`pattern`启用

## 语言分析器
elasticsearch提供了许多特定语言的分析器。通过配置"analyzer"为`english`或`french`等启用

## 指纹分析器
指纹分析器是创建应用于复制检测指纹的特殊的分析器。通过配置"analyzer"为`fingerprint`启用
