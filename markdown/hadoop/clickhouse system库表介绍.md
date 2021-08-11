# clickhouse system库表介绍
## query_log 

记录了查询开始时间，查询耗时以及查询语句，以及客户端版本、IP和端口等。非常适合用户慢查询的定位。

例如需要定位clickhouse 非常慢的时候，只要查询最近做了些什么。

```sql
SELECT *
FROM query_log
WHERE (event_time > toDate('2020-08-04 17:03:43')) AND (query_duration_ms > 30000)
```
查询八月4号以来，执行超过30s的查询。之后可以对比`read_rows`和 `result_rows` 查看是不是扫描了太多没有非命中的行。如果两个列之间差异巨大，建议给查询增加分区字段。如果两列的差异不大，就要观察一下是不是扫描的列太多，有没有返回无用数据等。query_log 表提供了clickhouse服务查询慢的定位需要的各项数据。

