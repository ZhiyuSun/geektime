## SQL语法基础篇

### 01丨了解SQL

我们可以把 SQL 语言按照功能划分成以下的 4 个部分：

- DDL，英文叫做 Data Definition Language，也就是数据定义语言，它用来定义我们的数据库对象，包括数据库、数据表和列。通过使用 DDL，我们可以创建，删除和修改数据库和表结构。
- DML，英文叫做 Data Manipulation Language，数据操作语言，我们用它操作和数据库相关的记录，比如增加、删除、修改数据表中的记录。
- DCL，英文叫做 Data Control Language，数据控制语言，我们用它来定义访问权限和安全级别。
- DQL，英文叫做 Data Query Language，数据查询语言，我们用它查询想要的记录，它是 SQL 语言的重中之重。在实际的业务中，我们绝大多数情况下都是在和查询打交道，因此学会编写正确且高效的查询语句，是学习的重点。

### 02丨DBMS的前世今生

### 03丨学会用数据库的方式思考SQL

### 04丨使用DDL创建数据库&数据表时需要注意什么？

### 05丨检索数据：你还在SELECT * 么？

SQL：SELECT '王者荣耀' as platform, name FROM heros

SQL：SELECT DISTINCT attack_range FROM heros

select 执行顺序

count(*)等

### 06丨数据过滤：SQL数据过滤都有哪些方法？

你能看出来通配符还是很有用的，尤其是在进行字符串匹配的时候。不过在实际操作过程中，我还是建议你尽量少用通配符，因为它需要消耗数据库更长的时间来进行匹配。即使你对 LIKE 检索的字段进行了索引，索引的价值也可能会失效。如果要让索引生效，那么 LIKE 后面就不能以（%）开头，比如使用LIKE '%太%'或LIKE '%太'的时候就会对全表进行扫描。如果使用LIKE '太%'，同时检索的字段进行了索引的时候，则不会进行全表扫描。

where和order by后面要加索引

### 07丨什么是SQL函数？为什么使用SQL函数可能会带来问题？

这里有一个有关命名规范的建议：
关键字和函数名称全部大写；
数据库名、表名、字段名称全部小写；
SQL 语句必须以分号结尾。


### 08丨什么是SQL的聚集函数，如何利用它们汇总表的数据？

你要记住，在 SELECT 查询中，关键字的顺序是不能颠倒的，它们的顺序是：

SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...

### 09丨子查询：子查询的种类都有哪些，如何提高子查询的性能？

按照子查询执行的次数，我们可以将子查询分成关联子查询和非关联子查询，其中非关联子查询与主查询的执行无关，只需要执行一次即可，而关联子查询，则需要将主查询的字段值传入子查询中进行关联查询。

### 10丨常用的SQL标准有哪些，在SQL92中是如何使用连接的？

等值连接

SELECT player_id, a.team_id, player_name, height, team_name FROM player AS a, team AS b WHERE a.team_id = b.team_id

非等值连接

SELECT p.player_name, p.height, h.height_level FROM player AS p, height_grades AS h WHERE p.height BETWEEN h.height_lowest AND h.height_highest

左连接

SELECT * FROM player LEFT JOIN team on player.team_id = team.team_id

右连接

SELECT * FROM player RIGHT JOIN team on player.team_id = team.team_id

自连接

SELECT b.player_name, b.height FROM player as a , player as b WHERE a.player_name = '布雷克-格里芬' and a.height < b.height

### 11丨SQL99是如何使用连接的，与SQL92的区别是什么？

完整的SELECT语句内部执行顺序是：
1、FROM子句组装数据（包括通过ON进行连接）
2、WHERE子句进行条件筛选
3、GROUP BY分组
4、使用聚集函数进行计算；
5、HAVING筛选分组；
6、计算所有的表达式；
7、SELECT 的字段；
8、ORDER BY排序
9、LIMIT筛选

### 12丨视图在SQL中的作用是什么，它是怎样工作的？

CREATE VIEW player_above_avg_height AS
SELECT player_id, height
FROM player
WHERE height > (SELECT AVG(height) from player)

SELECT * FROM player_above_avg_height

CREATE VIEW player_above_above_avg_height AS
SELECT player_id, height
FROM player
WHERE height > (SELECT AVG(height) from player_above_avg_height)

ALTER VIEW view_name AS
SELECT column1, column2
FROM table
WHERE condition

ALTER VIEW player_above_avg_height AS
SELECT player_id, player_name, height
FROM player
WHERE height > (SELECT AVG(height) from player)

DROP VIEW view_name

DROP VIEW player_above_avg_height

今天我讲解了视图的使用，包括创建，修改和删除视图。使用视图有很多好处，比如安全、简单清晰。

安全性：虚拟表是基于底层数据表的，我们在使用视图时，一般不会轻易通过视图对底层数据进行修改，即使是使用单表的视图，也会受到限制，比如计算字段，类型转换等是无法通过视图来对底层数据进行修改的，这也在一定程度上保证了数据表的数据安全性。同时，我们还可以针对不同用户开放不同的数据查询权限，比如人员薪酬是个敏感的字段，那么只给某个级别以上的人员开放，其他人的查询视图中则不提供这个字段。
简单清晰：视图是对 SQL 查询的封装，它可以将原本复杂的 SQL 查询简化，在编写好查询之后，我们就可以直接重用它而不必要知道基本的查询细节。同时我们还可以在视图之上再嵌套视图。这样就好比我们在进行模块化编程一样，不仅结构清晰，还提升了代码的复用率。

### 13丨什么是存储过程，在实际项目中用得多么？

### 14丨什么是事务处理，如何使用COMMIT和ROLLBACK进行操作？

A，也就是原子性（Atomicity）。原子的概念就是不可分割，你可以把它理解为组成物质的基本单位，也是我们进行数据处理操作的基本单位。
C，就是一致性（Consistency）。一致性指的就是数据库在进行事务操作后，会由原来的一致状态，变成另一种一致的状态。也就是说当事务提交后，或者当事务发生回滚后，数据库的完整性约束不能被破坏。
I，就是隔离性（Isolation）。它指的是每个事务都是彼此独立的，不会受到其他事务的执行影响。也就是说一个事务在提交之前，对其他事务都是不可见的。
最后一个 D，指的是持久性（Durability）。事务提交之后对数据的修改是持久性的，即使在系统出故障的情况下，比如系统崩溃或者存储介质发生故障，数据的修改依然是有效的。因为当事务完成，数据库的日志就会被更新，这时可以通过日志，让系统恢复到最后一次成功的更新状态。

ACID 可以说是事务的四大特性，在这四个特性中，原子性是基础，隔离性是手段，一致性是约束条件，而持久性是我们的目的。

### 19丨基础篇总结：如何理解查询优化、通配符以及存储过程？

