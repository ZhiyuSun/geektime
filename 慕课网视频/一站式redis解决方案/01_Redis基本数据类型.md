# Redis基本数据类型

熟悉5种基本数据结构的使用和应用场景是Redis知识最基础也是最重要的部分。

## 字符串（strings）

字符串是Redis最简单的储存类型，它存储的值可以是字符串、整数或者浮点数，对整个字符串或者字符串的其中一部分执行操作。对整数或者浮点数执行自增（increment）或者自减（decrement）操作。

Redis的字符串是一个由字节组成的序列，跟Java里面的ArrayList有点类似，采用预分配冗余空间的方式来减少内存的频繁分配，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。

字符串类型在工作中使用广泛，主要用于缓存数据，提高查询性能。比如存储登录用户信息、电商中存储商品信息、可以做计数器（想知道什么时候封锁一个IP地址（访问超过几次））等等。

set username zhangsan
get username
mset age 18 address bj
mget username age
incr num
decr num
incrby num 2
decrby num 3
del num

## 散列（hashes）

散列相当于Java中的HashMap
对比和java的hashset的区别

pic

hset key field value field value
hset userInfo username zhangsan age 18 address bj
hget userInfo username
hgetall userInfo
hlen userInfo
hincrby userInfo age 2
del userInfo
hdel userInfo age

## 列表（lists）

lpush student zhangsan lisi wangwu
rpush student tianqi
lpop student
rpop student
lrange student 0 1 (左右都包含)

## 集合（sets）

sadd nums 1 2 3
sadd nums 1 1 2 2 3 3
smembers nums
srem nums 2
spop nums
sadd nums1 1 2 3
sadd nums2 2 3 4
sinter nums1 nums2
sdiff nums1 nums2
sunion nums1 nums2

## 有序集合（sorted sets）

zadd key score value
zadd rank 66 zhangsan 88 lisi 77 wangwu 99 zhaoliu
zrange rank 0 3
zrangebyscore rank 77 99
zrem rank zhaoliu
zcard rank
zcount rank 77 88
zrank rank wangwu
zrevrank rank zhangsan 

### 二分查找

跳跃列表 skip lists
跳表由多条链表l0...lN以及下行指针构成

跳表和红黑树都是O(logn)，然后写法比较简单

## 统一登录认证项目

commons 一些通用的代码，错误码等等

ms-oauth2-server

SecurityConfiguration类
AuthorizationServerConfiguration 授权服务类

单点登录思想：在一个地方统一的进行登录验证

用户注册流程

