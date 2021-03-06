# SQL必知必会

## SQL性能优化

### 20丨当我们思考数据库调优的时候，都有哪些维度可以选择？

#### 数据库调优的目标
简单来说，数据库调优的目的就是要让数据库运行得更快，也就是说响应的时间更快，吞吐量更大。

方式：
- 用户的反馈
- 日志分析
- 服务器资源使用监控
- 数据库内部状况监控

#### 数据库调优的维度选择

第一步，选择适合的 DBMS
第二步，优化表设计
第三步，优化逻辑查询
第四步，优化物理查询

但你要知道索引不是万能的，我们需要根据实际情况来创建索引。那么都有哪些情况需要考虑呢？
- 如果数据重复度高，就不需要创建索引。通常在重复度超过 10% 的情况下，可以不创建这个字段的索引。比如性别这个字段（取值为男和女）。
- 要注意索引列的位置对索引使用的影响。比如我们在 WHERE 子句中对索引字段进行了表达式的计算，会造成这个字段的索引失效。
- 要注意联合索引对索引使用的影响。我们在创建联合索引的时候会对多个字段创建索引，这时索引的顺序就很重要了。比如我们对字段 x, y, z 创建了索引，那么顺序是 (x,y,z) 还是 (z,y,x)，在执行的时候就会存在差别。
- 要注意多个索引对索引使用的影响。索引不是越多越好，因为每个索引都需要存储空间，索引多也就意味着需要更多的存储空间。此外，过多的索引也会导致优化器在进行评估的时候增加了筛选出索引的计算时间，影响评估的效率。

第五步，使用 Redis 或 Memcached 作为缓存

第六步，库级优化

#### 我们该如何思考和分析数据库调优这件事

首先，选择比努力更重要。
另外，你可以把 SQL 查询优化分成两个部分，逻辑查询优化和物理查询优化。
最后，我们可以通过外援来增强数据库的性能。

### 21丨范式设计：数据表的范式有哪些，3NF指的是什么？

1NF 指的是数据库表中的任何属性都是原子性的，不可再分。
2NF 指的数据表里的非主属性都要和这个数据表的候选键有完全依赖关系。
3NF 在满足 2NF 的同时，对任何非主属性都不传递依赖于候选键

某种程度上 2NF 是对 1NF 原子性的升级。1NF 告诉我们字段属性需要是原子性的，而 2NF 告诉我们一张表就是一个独立的对象，也就是说一张表只表达一个意思。

1NF 需要保证表中每个属性都保持原子性；2NF 需要保证表中的非主属性与候选键完全依赖；3NF 需要保证表中的非主属性与候选键不存在传递依赖。

### 22丨反范式设计：3NF有什么不足，为什么有时候需要反范式设计？

我们在之前已经了解了越高阶的范式得到的数据表越多，数据冗余度越低。但有时候，我们在设计数据表的时候，还需要为了性能和读取效率违反范式化的原则。反范式就是相对范式化而言的，换句话说，就是允许少量的冗余，通过空间来换时间。

如果我们想对查询效率进行优化，有时候反范式优化也是一种优化思路。

那么反范式优化适用于哪些场景呢？

在现实生活中，我们经常需要一些冗余信息，比如订单中的收货人信息，包括姓名、电话和地址等。每次发生的订单收货信息都属于历史快照，需要进行保存，但用户可以随时修改自己的信息，这时保存这些冗余信息是非常有必要的。

当冗余信息有价值或者能大幅度提高查询效率的时候，我们就可以采取反范式的优化。

此外反范式优化也常用在数据仓库的设计中，因为数据仓库通常存储历史数据，对增删改的实时性要求不强，对历史数据的分析需求强。这时适当允许数据的冗余度，更方便进行数据分析。

### 23丨索引的概览：用还是不用索引，这是一个问题

#### 索引不是万能的，在有些情况下使用索引反而会让效率变低。

在数据表中的数据行数比较少的情况下，比如不到 1000 行，是不需要创建索引的。另外，当数据重复度大，比如高于 10% 的时候，也不需要对这个字段使用索引。我之前讲到过，如果是性别这个字段，就不需要对它创建索引。这是为什么呢？如果你想要在 100 万行数据中查找其中的 50 万行（比如性别为男的数据），一旦创建了索引，你需要先访问 50 万次索引，然后再访问 50 万次数据表，这样加起来的开销比不使用索引可能还要大。

#### 索引的种类有哪些？

从功能逻辑上说，索引主要有 4 种，分别是普通索引、唯一索引、主键索引和全文索引。

按照物理实现方式，索引可以分为 2 种：聚集索引和非聚集索引。我们也把非聚集索引称为二级索引或者辅助索引。

聚集索引与非聚集索引的原理不同，在使用上也有一些区别：
- 聚集索引的叶子节点存储的就是我们的数据记录，非聚集索引的叶子节点存储的是数据位置。非聚集索引不会影响数据表的物理存储顺序。
- 一个表只能有一个聚集索引，因为只能有一种排序存储的方式，但可以有多个非聚集索引，也就是多个索引目录提供数据检索。
- 使用聚集索引的时候，数据的查询效率高，但如果对数据进行插入，删除，更新等操作，效率会比非聚集索引低。

除了业务逻辑和物理实现方式，索引还可以按照字段个数进行划分，分成单一索引和联合索引。索引列为一列时为单一索引；多个列组合在一起创建的索引叫做联合索引。

这里需要说明的是联合索引存在最左匹配原则，也就是按照最左优先的方式进行索引的匹配。

使用索引可以帮助我们从海量的数据中快速定位想要查找的数据，不过索引也存在一些不足，比如占用存储空间、降低数据库写操作的性能等，如果有多个索引还会增加索引选择的时间。当我们使用索引时，需要平衡索引的利（提升查询效率）和弊（维护索引所需的代价）。

在实际工作中，我们还需要基于需求和数据本身的分布情况来确定是否使用索引，尽管索引不是万能的，但数据量大的时候不使用索引是不可想象的，毕竟索引的本质，是帮助我们提升数据检索的效率。

### 24丨索引的原理：我们为什么用B+树来做索引？

#### 如何评价索引的数据结构设计好坏

数据库服务器有两种存储介质，分别为硬盘和内存。内存属于临时存储，容量有限，而且当发生意外时（比如断电或者发生故障重启）会造成数据丢失；硬盘相当于永久存储介质，这也是为什么我们需要把数据保存到硬盘上。

虽然内存的读取速度很快，但我们还是需要将索引存放到硬盘上，这样的话，当我们在硬盘上进行查询时，也就产生了硬盘的 I/O 操作。相比于内存的存取来说，硬盘的 I/O 存取消耗的时间要高很多。我们通过索引来查找某行数据的时候，需要计算产生的磁盘 I/O 次数，当磁盘 I/O 次数越多，所消耗的时间也就越大。如果我们能让索引的数据结构尽量减少硬盘的 I/O 操作，所消耗的时间也就越小。

#### 什么是 B 树

如果用二叉树作为索引的实现结构，会让树变得很高，增加硬盘的 I/O 次数，影响数据查询的时间。因此一个节点就不能只有 2 个子节点，而应该允许有 M 个子节点 (M>2)。

B 树的出现就是为了解决这个问题，B 树的英文是 Balance Tree，也就是平衡的多路搜索树，它的高度远小于平衡二叉树的高度。在文件系统和数据库系统中的索引结构经常采用 B 树来实现。

一个 M 阶的 B 树（M>2）有以下的特性：
- 根节点的儿子数的范围是[2,M]。
- 每个中间节点包含 k-1 个关键字和 k 个孩子，孩子的数量 = 关键字的数量 +1，k 的取值范围为[ceil(M/2), M]。
- 叶子节点包括 k-1 个关键字（叶子节点没有孩子），k 的取值范围为[ceil(M/2), M]。
- 假设中间节点节点的关键字为：Key[1], Key[2], …, Key[k-1]，且关键字按照升序排序，即 Key[i]<Key[i+1]。此时 k-1 个关键字相当于划分了 k 个范围，也就是对应着 k 个指针，即为：P[1], P[2], …, P[k]，其中 P[1]指向关键字小于 Key[1]的子树，P[i]指向关键字属于 (Key[i-1], Key[i]) 的子树，P[k]指向关键字大于 Key[k-1]的子树。
- 所有叶子节点位于同一层。

你能看出来在 B 树的搜索过程中，我们比较的次数并不少，但如果把数据读取出来然后在内存中进行比较，这个时间就是可以忽略不计的。而读取磁盘块本身需要进行 I/O 操作，消耗的时间比在内存中进行比较所需要的时间要多，是数据查找用时的重要因素，B 树相比于平衡二叉树来说磁盘 I/O 操作要少，在数据查询中比平衡二叉树效率要高。

#### 什么是 B+ 树

有 k 个孩子的节点就有 k 个关键字。也就是孩子数量 = 关键字数，而 B 树中，孩子数量 = 关键字数 +1。
非叶子节点的关键字也会同时存在在子节点中，并且是在子节点中所有关键字的最大（或最小）。
非叶子节点仅用于索引，不保存数据记录，跟记录有关的信息都放在叶子节点中。而 B 树中，非叶子节点既保存索引，也保存数据记录。
所有关键字都在叶子节点出现，叶子节点构成一个有序链表，而且叶子节点本身按照关键字的大小从小到大顺序链接。

整个过程一共进行了 3 次 I/O 操作，看起来 B+ 树和 B 树的查询过程差不多，但是 B+ 树和 B 树有个根本的差异在于，B+ 树的中间节点并不直接存储数据。这样的好处都有什么呢？
首先，B+ 树查询效率更稳定。因为 B+ 树每次只有访问到叶子节点才能找到对应的数据，而在 B 树中，非叶子节点也会存储数据，这样就会造成查询效率不稳定的情况，有时候访问到了非叶子节点就可以找到关键字，而有时需要访问到叶子节点才能找到关键字。
其次，B+ 树的查询效率更高，这是因为通常 B+ 树比 B 树更矮胖（阶数更大，深度更低），查询所需要的磁盘 I/O 也会更少。同样的磁盘页大小，B+ 树可以存储更多的节点关键字。
不仅是对单个关键字的查询上，在查询范围上，B+ 树的效率也比 B 树高。这是因为所有关键字都出现在 B+ 树的叶子节点中，并通过有序链表进行了链接。而在 B 树中则需要通过中序遍历才能完成查询范围的查找，效率要低很多。

B 树和 B+ 树都可以作为索引的数据结构，在 MySQL 中采用的是 B+ 树，B+ 树在查询性能上更稳定，在磁盘页大小相同的情况下，树的构造更加矮胖，所需要进行的磁盘 I/O 次数更少，更适合进行关键字的范围查询。

### 25丨Hash索引的底层原理是什么？

### 26丨索引的使用原则：如何通过索引让SQL查询效率最大化？

#### 创建索引有哪些规律？

1. 字段的数值有唯一性的限制，比如用户名
2. 频繁作为 WHERE 查询条件的字段，尤其在数据表大的情况下
在数据量大的情况下，某个字段在 SQL 查询的 WHERE 条件中经常被使用到，那么就需要给这个字段创建索引了。创建普通索引就可以大幅提升数据查询的效率。
3. 需要经常 GROUP BY 和 ORDER BY 的列
4. UPDATE、DELETE 的 WHERE 条件列，一般也需要创建索引
5. DISTINCT 字段需要创建索引
6. 做多表 JOIN 连接操作时，创建索引需要注意以下的原则

#### 什么时候不需要创建索引

WHERE 条件（包括 GROUP BY、ORDER BY）里用不到的字段不需要创建索引
第二种情况是，如果表记录太少，比如少于 1000 个，那么是不需要创建索引的
第三种情况是，字段中如果有大量重复数据，也不用创建索引，比如性别字段。
最后一种情况是，频繁更新的字段不一定要创建索引

#### 什么情况下索引失效

1. 如果索引进行了表达式计算，则会失效
2. 如果对索引使用函数，也会造成失效
3. 在 WHERE 子句中，如果在 OR 前的条件列进行了索引，而在 OR 后的条件列没有进行索引，那么索引会失效。
4. 当我们使用 LIKE 进行模糊查询的时候，前面不能是 %
5. 索引列尽量设置为 NOT NULL 约束。
6. 我们在使用联合索引的时候要注意最左原则

### 27丨从数据页的角度理解B+树查询

今天我们学习了数据库中的基本存储单位，也就是页（Page），磁盘 I/O 都是基于页来进行读取的，在页之上还有区、段和表空间，它们都是更大的存储单位。我们在分配空间的时候会按照页为单位来进行分配，同一棵树上同一层的页与页之间采用双向链表，而在页里面，记录之间采用的单向链表的方式。

### 28丨从磁盘I/O的角度理解SQL查询的成本

数据页加载的三种方式

1. 内存读取
2. 随机读取
3. 顺序读取

上一节我们了解到了页是数据库存储的最小单位，这一节我们了解了在数据库中是如何加载使用页的。SQL 查询是一个动态的过程，从页加载的角度来看，我们可以得到以下两点结论：
- 位置决定效率。如果页就在数据库缓冲池中，那么效率是最高的，否则还需要从内存或者磁盘中进行读取，当然针对单个页的读取来说，如果页存在于内存中，会比在磁盘中读取效率高很多。
- 批量决定效率。如果我们从磁盘中对单一页进行随机读，那么效率是很低的（差不多 10ms），而采用顺序读取的方式，批量对页进行读取，平均一页的读取效率就会提升很多，甚至要快于单个页面在内存中的随机读取。

### 29丨为什么没有理想的索引？

### 30丨锁：悲观锁和乐观锁是什么？

### 32丨查询优化器是如何工作的？

