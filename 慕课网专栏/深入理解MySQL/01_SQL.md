# 开篇词

SQL 语句优化、索引原理、MySQL锁、事务、MySQL 安全、分库分表、读写分离、MySQL 操作规范等

书籍
如果你没有 MySQL 的基础，建议可以看下面两本书籍，看完之后，可以简单处理一些优化：
- 《MySQL 必知必会》：主要讲 SQL 的写法；
- 《深入浅出 MySQL》：比较全面的讲解了 MySQL 的基础知识，也涉及了一些优化。

如果已经对 MySQL 比较熟悉了，可以看下面的书籍，你会对索引和锁以及事务等有全新的看法：
- 《高性能 MySQL》：里面讲了很多 MySQL 优化技巧；
- 《MySQL 技术内幕》：讲解了很多 MySQL 原理，强力推荐给想深入学习 MySQL 的同学；
- 《MySQL 内核：InnoDB 存储引擎》：想深入研究 MySQL 内核及原理的可以看看；
- 《MySQL 运维内参》：对 MySQL 源码感兴趣，可以入手；
- 《MySQL Internals Manual》https://dev.mysql.com/doc/internals/en/ ；
- 《MySQL 5.7 Reference Manual》https://dev.mysql.com/doc/refman/5.7/en/ 。

# SQL

## 定位慢SQL

查看慢查询日志确定已经执行完的慢查询
show processlist 查看正在执行的慢查询

## exlain分析慢查询

Explain 可以获取 MySQL 中 SQL 语句的执行计划，比如语句是否使用了关联查询、是否使用了索引、扫描行数等。可以帮我们选择更好地索引和写出更优的 SQL 。使用方法：在查询语句前面加上 explain 运行就可以了。

``` SQL
CREATE DATABASE monki; /* 创建测试使用的database，名为muke */
use monki; /* 使用muke这个database */
drop table if exists t1; /* 如果表t1存在则删除表t1 */
CREATE TABLE `t1` ( /* 创建表t1 */
`id` int(11) NOT NULL auto_increment,
`a` int(11) DEFAULT NULL,
`b` int(11) DEFAULT NULL,
`create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '记录创建时间',
`update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '记录更新时间',
PRIMARY KEY (`id`),
KEY `idx_a` (`a`),
KEY `idx_b` (`b`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
drop procedure if exists insert_t1; /* 如果存在存储过程insert_t1，则删除 */
delimiter ;;
create procedure insert_t1() /* 创建存储过程insert_t1 */
begin
declare i int; /* 声明变量i */
set i=1; /* 设置i的初始值为1 */
while(i<=1000)do /* 对满足i<=1000的值进行while循环 */
insert into t1(a,b) values(i, i); /* 写入表t1中a、b两个字段，值都为i当前的值 */
set i=i+1; /* 将i加1 */
end while;
end;;
delimiter ; /* 创建批量写入1000条数据到表t1的存储过程insert_t1 */
call insert_t1(); /* 运行存储过程insert_t1 */
drop table if exists t2; /* 如果表t2存在则删除表t2 */
create table t2 like t1; /* 创建表t2，表结构与t1一致 */
insert into t2 select * from t1; /* 将表t1的数据导入到t2 */
```

``` SQL
explain select * from t1 where b=100;
```

列名解释
id 查询编号
select_type 查询类型：显示本行是简单还是复杂查询
table 涉及到的表
partitions 匹配的分区：查询将匹配记录所在的分区。仅当使用 partition 关键字时才显示该列。对于非分区表，该值为 NULL。
type 本次查询的表连接类型
possible_keys 可能选择的索引
key 实际选择的索引
key_len 被选择的索引长度：一般用于判断联合索引有多少列被选择了
ref 与索引比较的列
rows 预计需要扫描的行数，对 InnoDB 来说，这个值是估值，并不一定准确
filtered 按条件筛选的行的百分比
Extra 附加信息

今天我们分享了 show profile 和 trace 的使用方法，我们来对比一下三种分析 SQL 方法的特点：
- explain：获取 MySQL 中 SQL 语句的执行计划，比如语句是否使用了关联查询、是否使用了索引、扫描行数
等；
- profile：可以清楚了解到SQL到底慢在哪个环节；
- trace：查看优化器如何选择执行计划，获取每个可能的索引选择的代价。

## 不走索引的原因

数据表

``` SQL
CREATE TABLE `t1` ( /* 创建表t1 */
`id` int(11) NOT NULL AUTO_INCREMENT,
`a` varchar(20) DEFAULT NULL,
`b` int(20) DEFAULT NULL,
`c` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
PRIMARY KEY (`id`),
KEY `idx_a` (`a`) USING BTREE,
KEY `idx_b` (`b`) USING BTREE,
KEY `idx_c` (`c`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**1. 函数操作**

select * from t1 where date(c) ='2019-05-21';
改为：
select * from t1 where c>='2019-05-21 00:00:00' and c<='2019-05-21 23:59:59';

对索引字段做函数操作时，优化器会放弃使用索引。

**2. 隐式转换**

select * from t1 where a=1000;
等价于：
select * from t1 where cast(a as signed int) =1000;

改为：select * from t1 where a='1000';

**3. 模糊查询**

select * from t1 where a like '%1111%';
改为：
select * from t1 where a like '1111%';

如果条件只知道中间的值，需要模糊查询去查，那就建议使用ElasticSearch或其它搜索服务器。

**4. 范围查询**

select * from t1 where b>=1 and b <=2000;
改为：
select * from t1 where b>=1 and b <=1000;
select * from t1 where b>=1001 and b <=2000;

优化器会根据检索比例、表大小、I/O块大小等进行评估是否使用索引。比如单次查询的数据量过大，
优化器将不走索引。

**5. 计算操作**

select * from t1 where b-1 =1000;
改为：
select * from t1 where b =1000 + 1;

对索引字段做运算将使用不了索引。

一般需要对条件字段做计算时，建议通过程序代码实现，而不是通过MySQL实现。如果在MySQL中计算
的情况避免不了，那必须把计算放在等号后面。

总结
- 应该避免隐式转换
- like查询不能以%开头
- 范围查询时，包含的数据比例不能太大
- 不建议对条件字段做运算及函数操作

## 优化数据导入

有大批量导入时，推荐一条insert语句插入多行数据。

因为批量导入大部分时间耗费在客户端与服务端通信的时间，所以多条 insert 语句合并提交可以减少客户端与服务端通信的时间，并且合并提交还可以减少数据落盘的次数。

根据测试，总结了加快批量数据导入有如下方法：
- 一次插入多行的值；
- 关闭自动提交，多次插入数据的 SQL 一次提交；
- 调整参数，innodb_flush_log_at_trx_commit 和 sync_binlog 都设置为0（当然这种情况可能会丢数据）。

## 让order by、group by查询更快

### order by

MySQL 中的 Filesort 并不一定是在磁盘文件中进行排序的，也有可能在内存中排序，内存排序还是磁盘排序取决
于排序的数据大小和 sort_buffer_size 配置的大小。

#### order by优化技巧

**添加合适索引**

- 排序字段添加索引。排序字段添加索引。extra里的字段using index(有索引)，using filesort(无索引)
- 多个字段排序的情况，如果要通过添加索引优化，得注意排序字段的顺序与联合索引中列的顺序要一致。因此，如果多个字段排序，可以在多个排序字段上添加联合索引来优化排序语句。
- 先等值查询再排序的优化。对于先等值查询再排序的语句，可以通过在条件字段和排序字段添加联合索引来优化此类排序语句。

**去掉不必要的返回字段**

- 查询所有字段不走索引的原因是：扫描整个索引并查找到没索引的行的成本比扫描全表的成本更高，
所以优化器放弃使用索引。

**修改参数**

在本节一开始讲 order by 原理的时候，接触到两个跟排序有关的参数：max_length_for_sort_data、
sort_buffer_size。
- max_length_for_sort_data：如果觉得排序效率比较低，可以适当加大 max_length_for_sort_data 的值，让优化
器优先选择全字段排序。当然不能设置过大，可能会导致 CPU 利用率过低或者磁盘 I/O 过高；
- sort_buffer_size：适当加大 sort_buffer_size 的值，尽可能让排序在内存中完成。但不能设置过大，可能导致数
据库服务器 SWAP。

**几种无法利用索引排序的情况**

使用范围查询再排序
ASC 和 DESC 混合使用将无法使用索引

### group by

默认情况，会对 group by 字段排序，因此优化方式与 order by 基本一致，如果目的只是分组而不用排序，可以指
定 order by null 禁止排序。

## 分页查询

### 根据自增且连续主键排序的分页查询

select * from t1 limit 99000,2;
改写为：
select * from t1 where id >99000 limit 2;

这种改写得满足以下两个条件：
- 主键自增且连续
- 结果是按照主键排序的

### 查询根据非主键字段排序的分页查询

select * from t1 order by a limit 99000,2;

扫描整个索引并查找到没索引的行的成本比扫描全表的成本更高，所以优化器放弃使用索引。

其实关键是让排序时返回的字段尽可能少，所以可以让排序和分页操作先查出主键，然后根据主键查到对应的记录，SQL 改写如下：

select * from t1 f inner join (select id from t1 order by a limit 99000,2)g on f.id = g.id;

### 总结

本节讲到了两种分页查询场景的优化：
- 根据自增且连续主键排序的分页查询优化
- 查询根据非主键字段排序的分页查询优化

对于其它一些复杂的分页查询，也基本可以按照这两个思路去优化，尤其是第二种优化方式。第一种优化方式需要主键连续，而主键连续对于一个正常业务表来说可能有点困难，总会有些数据行删除的，但是占用了一个主键 id。

## join

### 关联字段添加索引

### 小表做驱动表

### 临时表

## count

当 count() 统计某一列时，比如 count(a)，a 表示列名，是不统计 null 的。

而 count(*) 无论是否包含空值，都会统计。

对于 MyISAM 引擎，如果没有 where 子句，也没检索其它列，那么 count(*) 将会非常快。因为 MyISAM 引擎会把表的总行数存在磁盘上。

而 InnoDB 并不会保留表中的行数，因为并发事务可能同时读取到不同的行数。所以执行 count(*) 时都是临时去计算的，会比 MyISAM 引擎慢很多。

对比 MyISAM 引擎和 InnoDB 引擎 count(*) 的区别，可以知道：
- MyISAM 会维护表的总行数，放在磁盘中，如果有 count(*) 的需求，直接返回这个数据
- 但是 InnoDB 就会去遍历普通索引树，计算表数据总量

在前面我们知道 count(*) 无论是否包含空值，所有结果都会统计。
而 count(1)中的 1 是恒真表达式，因此也会统计所有结果。
所以 count(1) 和 count(*) 统计结果没差别。

### count()优化

#### show table status

这个值是个估算值，可能与实际值相差 40% 到 50%。

#### 用 Redis 做计数器

通过 Redis 计数的方式，获取表的数据量比 show table status 准确，并且速度也比较快。

#### 增加计数表

计数这一步操作我们用 MySQL 中一张 InnoDB 表来代替。
而数据写入操作和计数操作都放在一个事务中

### 总结

本节首先讲解了 count(a) 和 count(*) 的区别。曾经遇到过这种情况，某个同事想要统计表的数据总量，因为考虑到某个字段（比如字段名就是 a 吧）有索引，就写成了 select count(a)，碰巧 a 字段存在 null，导致结果不准确。

然后对比了 MyISAM 引擎和 InnoDB 引擎 `count(*)`的区别，也说明了为什么 MyISAM 引擎执行`count(*)` 可以这么快，并提到了使用二级索引来处理 count() 语句比使用主键索引处理 `count(*)` 效率更高。还有就是 count(1) 和count`(*) `其实执行效率差不多。

后面提到几种优化 count() 的方式：
- show table status：能快速获取结果，但是结果不准确；
- 用 Redis 做计数器：能快速获取结果，比 show table status 结果准确，但是并发场景计数可能不准确；
- 增加 InnoDB 计数表：能快速获取结果，利用了事务特性确保了计数的准确，也是比较推荐的方法。