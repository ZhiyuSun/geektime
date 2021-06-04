# redis快速入门

高性能
- 底层C语言编写，内存数据库，通讯采用epoll非阻塞IO多路复用机制

线程安全
- 单线程原子操作，保证高并发情况下数据安全

功能丰富
- 数据结构
- 持久化，rdb,aof
- 主从
- 哨兵
- 集群
- 模块化

能够支持很多的互联网场景

学习redis难在哪？
- 理论+实践=实际应用

如何解决？项目！

## Redis介绍

### 特点

- 内存数据库（减少了IO操作），速度快（读每秒11w，写入每秒8w），也支持数据的持久化。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供Lists,hashes,sets,sorted sets等多种数据结构的存储
- Redis支持数据的备份（master-slave）与集群（分片存储），以及拥有哨兵监控机制。
- 支持事务

### 优势

- 性能极高-Redis能读的速度是110000次/s，写的速度是81000次/s
- 丰富的数据类型-Redis支持String、Lists、Hashes、Sets、Sorted Sets等数据类型操作
- 原子操作-Redis的所有操作都是原子性的，同时Redis还支持对几个操作合并后的原子性操作（事务）。
- 丰富的特性-Redis还支持publish/subscribe，通知，key过期等特性。

### Redis, memcached, Ecahce的区别

pic

### Redis的高并发

- 纯内存数据库，所以读取速度快。
- 使用非阻塞IO，IO多路复用，减少了线程切换时上下文的切换和竞争。
- 单线程模型，保证了每个操作的原子性，也减少了线程的上下文切换和竞争。
- 存储结构多样化，不同的数据结构对数据存储进行了优化，加快读取的速度。
- 采用自己实现的事件分离器，效率比较高，内部采用非阻塞的执行方式，吞吐能力比较大。

### Redis的单线程

原因
- 不需要各种锁的性能消耗
- 单线程多进程集群方案
- CPU消耗

优势
- 代码更清晰，处理逻辑更简单
- 不要去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗
- 不存在多进程或者多线程导致的切换而消耗CPU

弊端
- 无法发挥多核CPU性能，不过可以通过在单机开多个Redis实例来完善

### IO多路复用技术

传统模式，5个水龙头，5个管道
IO多路复用，5个水龙头，哪个出水了，用管子

采用网络IO多路复用技术来保证在多连接的时候，系统的高吞吐量（吞吐量：快速的写入和读取）。

## Redis安装

Redis6.0，支持多线程IO。多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。

视频里演示了很多在centos下面启动redis，略过。。

## Redis的配置

daemonize，是否是后台启动
bind
port
databases
save
dbfilename
dir
requirepass
maxclients
maxmemory

为了使桌面的客户端连上redis，配一下redis的bind和requirepass

## Redis的Java客户端

select 2
层级存储：set aa.bb.1 sun
keys *
info cpu
info repilication
flushall

Java客户端Jedis
连接池优化

Letture更推荐，是线程安全的

springboot集成Redis
springdata-redis

## 微服务架构

职责单一，隔离性强，开发简单

food-social-contact-parent
先搭建注册中心 ms-registry
然后搭建网关，new module ms-gateway

