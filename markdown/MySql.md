# MySql

## MySql引擎
### mysql支持的引擎及特性
#### MyISAM引擎

表级加锁。读取时，在表上获取共享锁（读锁）；写入时，在表上获取排他锁（写锁）。 

索引文件与数据文件单独存放；支持延迟更新索引；支持自动修复索引和手工修复索引。当使用表创建选项`DELAY_KEY_WRITE`创建的MyISAM表，在插入和修改之后不是将索引的改变写入磁盘，而是写入内存的键缓冲区中缓存索引的改变数据。只有在清理缓冲区和关闭表时，才将索引写入磁盘。如果服务器中间崩溃，索引将损坏，需要修复。
   
#### InnoDB
支持4种隔离级别，行级锁，所有索引包含主键，聚集索引。


## MySql索引
### 实现索引的数据结构划分

B-Tree索引
>每个叶子节点到根节点的距离是一样的，它的每一个叶子节点都包含了指向下一个节点的联接，以实现快速的范围遍历（实际上是B+Tree）。
>  
> 由于B-Tree是按照一定的次序组织索引，因此B-Tree索引能很好的用于全键值、键值范围或键前缀查找（最左前缀）。
>

Hash索引
>建立在hash表的基础之上。键是存储引擎计算出的被索引列的hash码，值是指向对应行的指针。 如果有多个行hash到一个槽位，索引就会把行指针以链表的方式保存。
>
>对索引中的每一列精确值查找有效
>

空间索引（R-Tree）
>R树是空间数据索引结构中重要的一种层次结构，目前已成为许多空间索引方法的基础，不少前沿的空间索引都适用到R树或者对R树的改良。其构建思想是以最小边界矩形（MBR）递归对空间数据集的空间按照"面积"规划进行划分。它的铁电如下
>
>1、R树中非叶子节点代表一个划分的空间区域，即一个矩形空间区域；
>
>2、R树中的叶子节点包含的矩形区域对应空间对象的MBR。 
>
>必须使用MySql GIS函数查找生效。

全文索引
> FULLTEXT 是一种特殊的索引。用于全文查找，使用倒排索引实现，本质上是一种特殊的两层结构的B树。


### 表内索引主次划分
主索引
>使用primary key指定的索引列

辅助索引
>使用key指定的索引列

### 表内索引与数据存放的维度划分
聚集索引
>索引和数据一起存放，查找数据时不需要取对应的数据行。如果不是索引键顺序插入，会导致大量数据移动，分页变稀疏，进而导致碎片。

非聚集索引
>索引和数据分开存放。叶子节点保存的引用的主键列，需要两次索引查找。

InnoDB的主索引是B-Tree索引，同时也是聚集索引。辅助索引是非聚集索引。
MyISAM主索引和辅助索引都是非聚集索引。

## MySql锁
两个或者多个事务在相同的资源集合上相互占用，并请求加锁，导致循环等待的现象。

### 产生锁的情况
对于支持表级别锁的表引擎。

事务1

```sql
start transaction;
update mytable1 set mycol = 1 where id = 1;
update mytable2 set mycol = 1 where id = 1;
commit;
```

事务2

```sql
start transaction;
update mytable2 set mycol = 1 where id = 1;
update mytable1 set mycol = 1 where id = 1;
commit;
```

对于支持行级别锁的表引擎。
事务1

```sql
start transaction;
update mytable1 set mycol = 1 where id = 1;
update mytable1 set mycol = 1 where id = 2;
commit;
```

事务2

```sql
start transaction;
update mytable1 set mycol = 1 where id = 2;
update mytable1 set mycol = 1 where id = 1;
commit;
```

解决方法：
1、等待，直到超时(innodb_lock_wait_timeout = 50s)

2、发起死锁检测，主动回滚一条事务，让其他事务继续执行。（innodb_deadlock_detect=on)
show engine innodb status 

## MySql优化
### 查询性能优化

### 系统优化

## MySql备份
### 备份

### 还原


## MySql事务特性

ANSI SQL隔离级别

- 读未提交（READ UNCOMMITED）可以读取其他事务未提交的结果。脏读、不可重复读、幻读

- 读已提交（READ COMMITED）不可重复读、幻读

- 可重复读（REPEATABLE READ）幻读

- 序列化（SERIALIZABLE）加锁读

MySql默认的隔离级别是可重复读，InnoDB通过多版本控制（MVCC）机制解决了幻读的问题。

MVCC实现步骤：
在每行上添加两个隐含列，分别为过期时间和创建时间。mysql每一次开启新事务时，系统版本号都会自动递增，过期时间和创建时间实际记录的是此次事务记录的当前系统版本号。

具体操作入下：
 
 * select
    > InnoDB只查找创建时间不晚于当前事务版本的数据行，确保当前事务读取的行都是在事务开始之前已经存在的,或者至少是当前事务创建和修改的。
    >
    >数据行的删除版本必须是为定义的，或是大于事务版本的，这保证了事务读取的行是在事务开始时未被删除的。
        
 * insert
    >InnoDB为每一个新增行记录当前系统版本号。
    
 * delete
    >InnoDB为每一个删除行行记录当前系统版本号，作为删除标识。
 
 * update
    >InnoDB为每一个需要更新的行建立一个新的行拷贝，并且为新的行拷贝记录当前系统版本号。同时也为更新前的旧行记录系统的版本号，作为旧行的删除版本标识。
 
