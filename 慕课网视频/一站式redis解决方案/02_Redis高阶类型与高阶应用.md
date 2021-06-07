# Redis高阶类型与高阶应用

## 代金券

表设计

限流，缓存，异步，分流

## 压力测试

jmeter

下载windows版

建立测试计划

5000个线程，2000个用户，出现超卖
10000个线程，1个用户，一个人买了4个

## Redis防止超卖

将活动写入Redis，通过Redis自减指令扣减库存

hutools工具集，转对象

opsForHash

redis里面其实有两步，先查后扣，结果还是不准确，这时候要用lua脚本，保证原子性

防止超卖：把库存写到redis里，redis先校验并扣减库存，再下单

## Redis限制一人一单

synchrozied只对同一个jvm里的多个线程有效

互斥，setnx

set test abc ex 60 nx

自旋锁，进不去转圈？

可重入锁

Redis可重入锁

