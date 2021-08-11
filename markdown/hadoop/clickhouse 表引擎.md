# clickhouse 表引擎

这篇文章整理于 [clickhouse官方文档](https://clickhouse.tech/docs/en/engines/table-engines/special/file/)，主要记录了clickhouse中支持的表引擎，以及各个引擎的特性。 
## clickhouse支持的表引擎

以下是clickhouse支持的四种类型的表引擎： 

- MergeTree系列
- Log系列
- 集成表引擎
- 特殊表引擎

MergeTree系列的表引擎是clickhouse


### MergeTree系列

MergeTree系列的引擎为了向表中插入大量数据而设计的。数据被一块块的快速写入，然后在后台按照规则合并块。

需要特性： 

- 存储的数据按照住建排序

   支持创建少量的稀疏索引帮助快速的查找数据

- 可以使用分区字段分区
 
   相比于普通的查询，操作分区字段在同样的数据上得到同样的结果会更高效。clickhouse会在指定了分区的查询中自动切断分区数据。这样也会提高查询效率。 
 
- 支持数据复制

   `ReplicatedMergeTree ` 提供了数据复制支持。 

- 支持数据抽样

   你可以在表中设置数据抽样方法。
   
### 创建表

创建表的语法： 

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
ORDER BY  expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...]
```

关键字说明：

- ENGINE： 
   表引擎的名字。例如 `ENGINE = MergeTree()` MergeTree表引擎没有参数。 
   
- ORDER BY： 
   列名称或任意表达式的元组。
   
- PARTITION BY： 可选。 分区字段，

