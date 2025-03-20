# Redis基础

## [1. 什么是 Redis？](https://javaguide.cn/database/redis/redis-questions-01.html#什么是-redis)

基于 C 语言开发的开源 NoSQL 数据库，保存在内存中的（内存数据库，支持持久化），因此读写速度非常快，被广泛应用于分布式缓存方向。Redis 存储的是 KV 键值对数据。

## [2. Redis 为什么这么快？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-为什么这么快)

1. Redis **基于内存**，内存的访问速度比磁盘快很多；
2. Redis 内置了多种**优化过后的数据类型/结构实现**，性能非常高。
3. Redis 基于 Reactor 模式设计开发了一套高效的事件处理模型，主要是单线程事件循环和 IO 多路复用（Redis 线程模式后面会详细介绍到）；
4. Redis 通信协议实现简单且解析高效

## [3. 除了 Redis，你还知道其他分布式缓存方案吗？](https://javaguide.cn/database/redis/redis-questions-01.html#除了-redis-你还知道其他分布式缓存方案吗)

Memcached 是分布式缓存最开始兴起的那会，比较常用的。后来，随着 Redis 的发展，大家慢慢都转而使用更加强大的 Redis 了。

分布式缓存首选 Redis ，毕竟经过这么多年的生考验，生态也这么优秀，资料也很全面！

## [4. 为什么要用 Redis？](https://javaguide.cn/database/redis/redis-questions-01.html#为什么要用-redis)

**1、访问速度更快**

传统数据库数据保存在磁盘，而 Redis 基于内存，内存的访问速度比磁盘快很多。引入 Redis 之后，我们可以把一些高频访问的数据放到 Redis 中，这样下次就可以直接从内存中读取，速度可以提升几十倍甚至上百倍。

**2、高并发**

一般像 MySQL 这类的数据库的 QPS 大概都在 4k 左右（4 核 8g） ，但是使用 Redis 缓存之后很容易达到 5w+，甚至能达到 10w+（就单机 Redis 的情况，Redis 集群的话会更高）。

**3、功能全面**

Redis 除了可以用作缓存之外，还可以用于分布式锁、限流、消息队列、延时队列等场景，功能强大！

## 5. 3种常用的缓存读写策略详解

### [5.1 Cache Aside Pattern（旁路缓存模式）](https://javaguide.cn/database/redis/3-commonly-used-cache-read-and-write-strategies.html#cache-aside-pattern-旁路缓存模式)

由缓存调用者，在更新数据库的同时更新缓存。

**在写数据的过程中，可以先删除 cache ，后更新 db 么？**

不行，会出现数据不一致，A删除缓存->B查询缓存未命中，查询数据库中的旧数据，把旧数据写入缓存->A更新数据库为新数据。

**反过来呢？先更新db在删除cache。**

出现数据不一致的概率很低，因为缓存的写入速度是比数据库的写入速度快很多，需要在很短的时间内更新数据库才会出现缓存不一致。

![image-20241016164359215](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502030832310.png)

缺点：

**缺陷 1：首次请求数据一定不在 cache 的问题**

解决办法：可以将热点数据可以提前放入 cache 中。

**缺陷 2：写操作比较频繁的话导致 cache 中的数据会被频繁被删除，这样会影响缓存命中率 。**

- 数据库和缓存数据强一致场景：更新 db 的时候同样更新 cache，不过我们需要加一个锁/分布式锁来保证更新 cache 的时候不存在线程安全问题。
- 可以短暂地允许数据库和缓存数据不一致的场景：更新 db 的时候同样更新 cache，但是给缓存加一个比较短的过期时间，这样的话就可以保证即使数据不一致的话影响也比较小。

### [5.2 Read/Write Through Pattern（读写穿透）](https://javaguide.cn/database/redis/3-commonly-used-cache-read-and-write-strategies.html#read-write-through-pattern-读写穿透)

缓存与数据库整合为一个服务，由服务来维护一致性。调用者调用该服务，无需关心缓存一致性问题。

服务端把 cache 视为主要数据存储，从中读取数据并将数据写入其中。

cache 服务负责将此数据读取和写入 db，从而减轻了应用程序的职责。

### [5.3 Write Behind Pattern（异步缓存写入）](https://javaguide.cn/database/redis/3-commonly-used-cache-read-and-write-strategies.html#write-behind-pattern-异步缓存写入)

调用者只操作缓存，由其它线程异步的将缓存数据持久化到数据库，保证最终一致。

这种策略在我们平时开发过程中也非常非常少见，但是不代表它的应用场景少，比如MySQL 的 Innodb Buffer Pool 机制用到了这种策略。

# [Redis 应用](https://javaguide.cn/database/redis/redis-questions-01.html#redis-应用)

## [1. Redis 除了做缓存，还能做什么？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-除了做缓存-还能做什么)

**分布式锁**：通过 Redis 来做分布式锁是一种比较常见的方式。通常情况下，我们都是基于 Redisson 来实现分布式锁。

**消息队列**：Redis 自带的 List 数据结构可以作为一个简单的队列使用。Redis 5.0 中增加的 Stream 类型的数据结构更加适合用来做消息队列。它比较类似于 Kafka，有主题和消费组的概念，支持消息持久化以及 ACK 机制。

**分布式 Session** ：利用 String 或者 Hash 数据类型保存 Session 数据，所有的服务器都可以访问。

## [2. 如何基于 Redis 实现分布式锁？](https://javaguide.cn/database/redis/redis-questions-01.html#如何基于-redis-实现分布式锁)

基础的，用Lua脚本配合setnx命令，删除的时候先判断是不是自己的锁。

## [3. Redis 可以做消息队列么？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-可以做消息队列么)

**可以是可以，但不建议使用 Redis 来做消息队列。和专业的消息队列相比，还是有很多欠缺的地方。**

## [4. Redis 可以做搜索引擎么？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-可以做搜索引擎么)

Redis 是可以实现全文搜索引擎功能的，需要借助 **RediSearch** ，这是一个基于 Redis 的搜索引擎模块。

## [5. 如何基于 Redis 实现延时任务？](https://javaguide.cn/database/redis/redis-questions-01.html#如何基于-redis-实现延时任务)

订单在 10 分钟后未支付就失效，如何用 Redis 实现？

![image-20250203091825568](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502030918600.png)

# [Redis 数据类型](https://javaguide.cn/database/redis/redis-questions-01.html#redis-数据类型)

## [1. Redis 数据类型有哪些？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-常用的数据类型有哪些)

### 1.0 基本概念

**五种基本类型：**

String（字符串）、List（列表）、Set（集合）、Hash（散列）、Zset（有序集合）。

**八种数据结构：**

简单动态字符串（SDS）、LinkedList（双向链表）、Dict（哈希表/字典）、SkipList（跳跃表）、Intset（整数集合）、ZipList（压缩列表）、QuickList（快速列表）。

### 1.1 底层实现

![image-20250203115114493](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502031151527.png)

| 数据结构     | 查询性能 | 插入/删除性能 | 内存占用       | 适用场景             |
| ------------ | -------- | ------------- | -------------- | -------------------- |
| **ZipList**  | O(N)     | O(N)          | 低（连续存储） | 小规模数据，节省内存 |
| **SkipList** | O(log N) | O(log N)      | 高（维护索引） | 大规模数据，高效查询 |



**ZipList（压缩列表）**

`ZipList` 是一种紧凑的、连续内存存储的数据结构，适用于存储较小的哈希。当哈希中的元素较少且每个元素的键和值都比较小时，Redis 会使用 `ZipList` 来存储哈希。

- **内存紧凑**：`ZipList` 将所有元素连续存储在一块内存中，减少了内存碎片和指针的开销。
- **元素限制**：`ZipList` 适用于元素数量较少且元素大小较小的场景。当元素数量或大小超过一定阈值时，Redis 会将 `ZipList` 转换为 `Dict`。
- **时间复杂度**：由于 `ZipList` 是线性结构，查找、插入和删除操作的时间复杂度为 O(n)，但由于元素数量较少，实际性能影响不大。

**Dict：**

`Dict` 是 Redis 中最常用的数据结构之一，它是一个基于哈希表的键值对存储结构。当哈希中的元素较多时，Redis 会使用 `Dict` 来存储哈希。

- **哈希表**：`Dict` 使用哈希表来实现，哈希表的每个桶（bucket）存储一个键值对。Redis 使用链地址法来解决哈希冲突。
- **动态扩容**：当哈希表中的元素数量超过一定阈值时，Redis 会自动对哈希表进行扩容，以减少哈希冲突，保证查询效率。
- **时间复杂度**：在平均情况下，`Dict` 的插入、删除和查找操作的时间复杂度都是 O(1)。

**IntSet（整数集合）**

当集合中的元素都是整数且元素数量较少时，Redis 会使用 `IntSet` 来存储集合。

- **有序数组**：`IntSet` 是一个有序的整数数组，元素按照从小到大的顺序存储。这种结构在存储整数时非常高效。
- **内存紧凑**：`IntSet` 将所有整数连续存储在一块内存中，减少了内存碎片和指针的开销。
- **元素限制**：`IntSet` 适用于元素数量较少且元素为整数的场景。当元素数量超过一定阈值或元素类型不是整数时，Redis 会将 `IntSet` 转换为 `Dict`。
- **时间复杂度**：由于 `IntSet` 是有序数组，查找操作可以使用二分查找，时间复杂度为 O(log n)。插入和删除操作的时间复杂度为 O(n)，但由于元素数量较少，实际性能影响不大。

**QuickList**

在早期版本的 Redis 中，列表数据结构有两种实现方式：

- **双向链表**：支持高效的插入和删除操作，但每个节点都需要存储前后指针，内存开销较大。
- **压缩列表（ZipList）**：内存紧凑，适合存储较小的列表，但当列表元素较多或较大时，性能会下降。

为了克服这两种结构的局限性，Redis 引入了 `QuickList`，它将多个 `ZipList` 通过双向链表连接起来，既保留了 `ZipList` 的内存紧凑性，又通过双向链表支持高效的插入和删除操作

`QuickList` 的整体结构是一个双向链表，其中每个节点都是一个 `ZipList`。

通过这种方式，`QuickList` 可以动态地调整每个 `ZipList` 的大小，以适应不同的使用场景。

![image-20250211230700205](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502112307704.png)

### [1.2 String（字符串）](https://javaguide.cn/database/redis/redis-data-structures-01.html#string-字符串)

虽然 Redis 是用 C 语言写的，但是 Redis 并没有使用 C 的字符串表示，而是自己构建了一种 **简单动态字符串**（Simple Dynamic String，**SDS**）。

相比于 C 的原生字符串，Redis 的 SDS 不光可以保存文本数据还可以保存二进制数据。

并且获取字符串长度复杂度为 O(1)（C 字符串为 O(N)）,除此之外，Redis 的 SDS API 是安全的，不会造成缓冲区溢出。



**应用场景：**

**需要存储常规数据的场景**

- 举例：缓存 Session、Token、图片地址、序列化后的对象(相比较于 Hash 存储更节省内存)。
- 相关命令：`SET`、`GET`。

**需要计数的场景**

- 举例：用户单位时间的请求数（简单限流可以用到）、页面单位时间的访问数。
- 相关命令：`SET`、`GET`、 `INCR`、`DECR` 。

**分布式锁**

利用 `SETNX key value` 命令可以实现一个最简易的分布式锁（存在一些缺陷，通常不建议这样实现分布式锁）。

### [1.3 List（列表）](https://javaguide.cn/database/redis/redis-data-structures-01.html#list-列表)

Redis 的 List 的实现为一个 **双向链表**。	

**应用场景：**

- 最新文章、最新动态。
- 相关命令：`LPUSH`、`LRANGE`。

### [1.4 Hash（哈希）](https://javaguide.cn/database/redis/redis-data-structures-01.html#hash-哈希)

Hash 类似于 JDK1.8 前的 `HashMap`，内部实现也差不多(数组 + 链表)。不过，Redis 的 Hash 做了更多优化。

**应用场景：**

对象数据存储场景：用户信息、商品信息、文章信息、购物车信息。

### [1.5 Set（集合）](https://javaguide.cn/database/redis/redis-data-structures-01.html#set-集合)

类似于 Java 中的 `HashSet`。

你可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。这样的话，Set 可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。

**应用场景：**

文章的点赞信息、用户的关注信息

### [1.6 Sorted Set（有序集合）](https://javaguide.cn/database/redis/redis-data-structures-01.html#sorted-set-有序集合)

类似于 Set，但和 Set 相比，Sorted Set 增加了一个权重参数 `score`，使得集合中的元素能够按 `score` 进行有序排列，还可以通过 `score` 的范围来获取元素的列表。

**应用场景：**

各种排行榜、点赞量排行榜、时间顺序排行。	

### 1.7 特殊数据类型

HyperLogLog（基数统计）、Bitmap （位图）、Geospatial (地理位置)

## [2. String 还是 Hash 存储对象数据更好呢？](https://javaguide.cn/database/redis/redis-questions-01.html#string-还是-hash-存储对象数据更好呢)

- 在绝大多数情况下，**String** 更适合存储对象数据，尤其是当对象结构简单且整体读写是主要操作时。
- 如果你需要频繁操作对象的部分字段或节省内存，**Hash** 可能是更好的选择。因为Hash 是对对象的每个字段单独存储，可以获取部分字段的信息，也可以修改或者添加部分字段，节省网络流量

## [3. String 的底层实现是什么？](https://javaguide.cn/database/redis/redis-questions-01.html#string-的底层实现是什么)

Redis 是基于 C 语言编写的，但 Redis 的 String 类型的底层实现并不是 C 语言中的字符串（即以空字符 `\0` 结尾的字符数组），而是自己编写了 [SDS](https://github.com/antirez/sds)（Simple Dynamic String，简单动态字符串） 来作为底层实现。

~~~java
/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
~~~

SDS 共有五种实现方式 SDS_TYPE_5（并未用到）、SDS_TYPE_8、SDS_TYPE_16、SDS_TYPE_32、SDS_TYPE_64，其中只有后四种实际用到。Redis 会根据初始化的长度决定使用哪种类型，从而减少内存的使用。

![image-20250203143550905](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502031435940.png)

SDS 相比于 C 语言中的字符串有如下提升：

1. **可以避免缓冲区溢出**：C 语言中的字符串被修改（比如拼接）时，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出。SDS 被修改时，会先根据 len 属性检查空间大小是否满足要求，如果不满足，则先扩展至所需大小再进行修改操作。
2. **获取字符串长度的复杂度较低**：C 语言中的字符串的长度通常是经过遍历计数来实现的，时间复杂度为 O(n)。SDS 的长度获取直接读取 len 属性即可，时间复杂度为 O(1)。
3. **减少内存分配次数**：为了避免修改（增加/减少）字符串时，每次都需要重新分配内存（C 语言的字符串是这样的），SDS 实现了空间预分配和惰性空间释放两种优化策略。当 SDS 需要增加字符串时，Redis 会为 SDS 分配好内存，并且根据特定的算法分配多余的内存，这样可以减少连续执行字符串增长操作所需的内存重分配次数。当 SDS 需要减少字符串时，这部分内存不会立即被回收，会被记录下来，等待后续使用（支持手动释放，有对应的 API）。
4. **二进制安全**：C 语言中的字符串以空字符 `\0` 作为字符串结束的标识，这存在一些问题，像一些二进制文件（比如图片、视频、音频）就可能包括空字符，C 字符串无法正确保存。SDS 使用 len 属性判断字符串是否结束，不存在这个问题。

## [4. 购物车信息用 String 还是 Hash 存储更好呢?](https://javaguide.cn/database/redis/redis-questions-01.html#购物车信息用-string-还是-hash-存储更好呢)

由于购物车中的商品频繁修改和变动，购物车信息建议使用 Hash 存储：

![Hash维护简单的购物车信息](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502031439090.png)

## [5. 使用 Redis 实现一个排行榜怎么做？](https://javaguide.cn/database/redis/redis-questions-01.html#使用-redis-实现一个排行榜怎么做)

Redis 中有一个叫做 `Sorted Set` （有序集合）的数据类型经常被用在各种排行榜的场景，比如直播间送礼物的排行榜、朋友圈的微信步数排行榜、王者荣耀中的段位排行榜、话题热度排行榜等等。

相关的一些 Redis 命令: `ZRANGE` (从小到大排序)、 `ZREVRANGE` （从大到小排序）、`ZREVRANK` (指定元素排名)。

## [6. Redis 的有序集合底层为什么要用跳表，而不用平衡树、红黑树或者 B+树？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-的有序集合底层为什么要用跳表-而不用平衡树、红黑树或者-b-树)

跳表：多层链表，最底层存放全部元素，上面的是索引。

![image-20250203145033433](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502031450465.png)

- 平衡树 vs 跳表：平衡树的插入、删除和查询的时间复杂度和跳表一样都是 **O(log n)**。对于范围查询来说，平衡树也可以通过中序遍历的方式达到和跳表一样的效果。**但是它的每一次插入或者删除操作都需要保证整颗树左右节点的绝对平衡，只要不平衡就要通过旋转操作来保持平衡，这个过程是比较耗时的**。跳表诞生的初衷就是为了克服平衡树的一些缺点。**跳表使用概率平衡而不是严格强制的平衡，因此，跳表中的插入和删除算法比平衡树的等效算法简单得多，速度也快得多**。
- 红黑树 vs 跳表：相比较于红黑树来说，**跳表的实现也更简单一些，不需要通过旋转和染色（红黑变换）来保证黑平衡**。并且，**按照区间来查找数据这个操作，红黑树的效率没有跳表高**。
- B+树 vs 跳表：B+树更适合作为数据库和文件系统中常用的索引结构之一，它的核心思想是通过可能少的 IO 定位到尽可能多的索引来获得查询数据。对于 Redis 这种内存数据库来说，它对这些并不感冒，因为 **Redis 作为内存数据库它不可能存储大量的数据，所以对于索引不需要通过 B+树这种方式进行维护**，只需**按照概率进行随机维护即可**，节约内存。而且使用跳表实现 zset 时相较前者来说更简单一些，在进行插入时只需通过索引将数据插入到链表中合适的位置**再随机维护一定高度的索引即可**，也不需要像 B+树那样插入时发现失衡时还需要对节点分裂与合并。

## [7. Set 的应用场景是什么？](https://javaguide.cn/database/redis/redis-questions-01.html#set-的应用场景是什么)

- 存放的数据不能重复的场景：文章点赞、动态点赞等等。
- 需要获取多个数据源交集、并集和差集的场景：共同好友(交集)、共同粉丝(交集)、共同关注(交集)、好友推荐（差集）、音乐推荐（差集）、订阅号推荐（差集+交集） 等等。
- 需要随机获取数据源中的元素的场景：抽奖系统、随机点名等等。

## [8. 使用 Set 实现抽奖系统怎么做？](https://javaguide.cn/database/redis/redis-questions-01.html#使用-set-实现抽奖系统怎么做)

- `SADD key member1 member2 ...`：向指定集合添加一个或多个元素。
- `SPOP key count`：随机移除并获取指定集合中一个或多个元素，适合不允许重复中奖的场景。
- `SRANDMEMBER key count` : 随机获取指定集合中指定数量的元素，适合允许重复中奖的场景。

## [9. 使用 Bitmap 统计活跃用户怎么做？](https://javaguide.cn/database/redis/redis-questions-01.html#使用-bitmap-统计活跃用户怎么做)

见javaguide。

## [10. 使用 HyperLogLog 统计页面 UV 怎么做？](https://javaguide.cn/database/redis/redis-questions-01.html#使用-hyperloglog-统计页面-uv-怎么做)

- `PFADD key element1 element2 ...`：添加一个或多个元素到 HyperLogLog 中。
- `PFCOUNT key1 key2`：获取一个或者多个 HyperLogLog 的唯一计数。

# Redis持久化机制详解

见Redis高级和原理。

# [Redis 线程模型](https://javaguide.cn/database/redis/redis-questions-01.html#redis-线程模型-重要)

## [1. Redis 单线程模型了解吗？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-单线程模型了解吗)

在 Redis 6.0 之前，Redis 是典型的单线程模型，Redis 的核心操作（执行命令）（数据读写、过期键处理）是由一个主线程完成的。

从 Redis 6.0 开始，Redis 引入了多线程支持，但核心逻辑仍然是单线程的。多线程主要用于 **网络 I/O** 和 **部分后台任务**。

1. **网络 I/O 多线程**：
   - Redis 6.0 引入了多线程来处理网络 I/O（读取请求和发送响应）。
   - 主线程仍然负责命令的执行，但网络数据的读取和写入可以由多个 I/O 线程并行处理。
   - 这样可以减轻主线程的负担，提升高并发场景下的性能。
2. **后台任务多线程**：
   - 一些耗时的后台任务（如持久化、键过期删除等）可以使用多线程来执行。
   - 例如，Redis 6.0 中，`UNLINK` 命令用于异步删除大键（DEL的异步版本），避免阻塞主线程。



**既然是单线程，那怎么监听大量的客户端连接呢？**

Redis 通过 **IO 多路复用程序** 来监听来自客户端的大量连接（或者说是监听多个 socket），它会将感兴趣的事件及类型（读、写）注册到内核中并监听每个事件是否发生。

**I/O 多路复用技术的使用让 Redis 不需要额外创建多余的线程来监听客户端的大量连接，降低了资源的消耗**。

## [2. Redis6.0 之前为什么不使用多线程？](https://javaguide.cn/database/redis/redis-questions-01.html#redis6-0-之前为什么不使用多线程)

- 单线程编程容易并且更容易维护；
- Redis 的性能瓶颈不在 CPU ，主要在内存和网络；
- 多线程就会存在死锁、线程上下文切换等问题，甚至会影响性能。

## [3. Redis6.0 之后为何引入了多线程？](https://javaguide.cn/database/redis/redis-questions-01.html#redis6-0-之后为何引入了多线程)

**Redis6.0 引入多线程主要是为了提高网络 IO 读写性能**，因为这个算是 Redis 中的一个**性能瓶颈**。

虽然，Redis6.0 引入了多线程，但是 Redis 的多线程只是在网络数据的读写这类耗时操作上使用了，执行命令仍然是单线程顺序执行。因此，你也不需要担心线程安全问题。

## [4. Redis 后台线程了解吗？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-后台线程了解吗)

- 通过 `bio_close_file` 后台线程来释放 AOF / RDB 等过程中产生的临时文件资源。
- 通过 `bio_aof_fsync` 后台线程调用 `fsync` 函数将系统内核缓冲区还未同步到到磁盘的数据强制刷到磁盘（ AOF 文件）。
- 通过 `bio_lazy_free`后台线程释放大对象（已删除）占用的内存空间.

# [Redis 内存管理](https://javaguide.cn/database/redis/redis-questions-01.html#redis-内存管理)

## [1. Redis 给缓存数据设置过期时间有什么用？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-给缓存数据设置过期时间有什么用)

通过设置合理的过期时间，Redis 会自动删除暂时不需要的数据，为新的缓存数据腾出空间。

Redis 中除了字符串类型有自己独有设置过期时间的命令 `setex` 外，其他方法都需要依靠 `expire` 命令来设置过期时间 。另外， `persist` 命令可以移除一个键的过期时间。

## [2. Redis 是如何判断数据是否过期的呢？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-是如何判断数据是否过期的呢)

Redis 通过一个叫做过期字典（可以看作是 hash 表）来保存数据过期的时间。过期字典的键指向 Redis 数据库中的某个 key(键)，过期字典的值是一个 long long 类型的整数，这个整数保存了 key 所指向的数据库键的过期时间（毫秒精度的 UNIX 时间戳）。

## [3. Redis 过期 key 删除策略了解么？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-过期-key-删除策略了解么)

如果假设你设置了一批 key 只能存活 1 分钟，那么 1 分钟后，Redis 是怎么对这批 key 进行删除的呢？

常用的过期数据的删除策略就下面这几种：

1. **惰性删除**：只会在取出/查询 key 的时候才对数据进行过期检查。这种方式对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。
2. **定期删除**：周期性地随机从设置了过期时间的 key 中抽查一批，然后逐个检查这些 key 是否过期，过期就删除 key。相比于惰性删除，定期删除对内存更友好，对 CPU 不太友好。
3. **延迟队列**：把设置过期时间的 key 放到一个延迟队列里，到期之后就删除 key。这种方式可以保证每个过期 key 都能被删除，但维护延迟队列太麻烦，队列本身也要占用资源。
4. **定时删除**：每个设置了过期时间的 key 都会在设置的时间到达时立即被删除。这种方法可以确保内存中不会有过期的键，但是它对 CPU 的压力最大，因为它需要为每个键都设置一个定时器。

Redis 采用的是 **定期删除+惰性/懒汉式删除** 结合的策略，这也是大部分缓存框架的选择。定期删除对内存更加友好，惰性删除对 CPU 更加友好。两者各有千秋，结合起来使用既能兼顾 CPU 友好，又能兼顾内存友好。

Redis 的定期删除过程是随机的（**周期性地随机从设置了过期时间的 key 中抽查一批**），所以并不保证所有过期键都会被立即删除。这也就解释了为什么有的 key 过期了，并没有被删除。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。

另外，定期删除还会受到执行时间和过期 key 的比例的影响：

- 执行时间已经超过了阈值，那么就中断这一次定期删除循环，以避免使用过多的 CPU 时间。
- 如果这一批过期的 key 比例超过一个比例，就会重复执行此删除流程，以更积极地清理过期 key。相应地，如果过期的 key 比例低于这个比例，就会中断这一次定期删除循环，避免做过多的工作而获得很少的内存回收。



**为什么定期删除不是把所有过期 key 都删除呢？**

这样会对性能造成太大的影响。如果我们 key 数量非常庞大的话，挨个遍历检查是非常耗时的，会严重影响性能。Redis 设计这种策略的目的是为了平衡内存和性能。

**为什么 key 过期之后不立马把它删掉呢？这样不是会浪费很多内存空间吗？**

因为不太好办到，或者说这种删除方式的成本太高了。假如我们使用延迟队列作为删除策略，这样存在下面这些问题：

1. 队列本身的开销可能很大：key 多的情况下，一个延迟队列可能无法容纳。
2. 维护延迟队列太麻烦：修改 key 的过期时间就需要调整期在延迟队列中的位置，并且，还需要引入并发控制。

## [4. 大量 key 集中过期怎么办？](https://javaguide.cn/database/redis/redis-questions-01.html#大量-key-集中过期怎么办)

- **请求延迟增加：** Redis 在处理过期 key 时需要消耗 CPU 资源，如果过期 key 数量庞大，会导致 Redis 实例的 CPU 占用率升高，进而影响其他请求的处理速度，造成延迟增加。
- **内存占用过高：** 过期的 key 虽然已经失效，但在 Redis 真正删除它们之前，仍然会占用内存空间。如果过期 key 没有及时清理，可能会导致内存占用过高，甚至引发内存溢出。

解决方法：

1. **尽量避免 key 集中过期**: 在设置键的过期时间时尽量随机一点。
2. **开启 lazy free 机制**: 修改 `redis.conf` 配置文件，将 `lazyfree-lazy-expire` 参数设置为 `yes`，即可开启 lazy free 机制。开启 lazy free 机制后，Redis 会在后台异步删除过期的 key，不会阻塞主线程的运行，从而降低对 Redis 性能的影响。

## [5. Redis 内存淘汰策略了解么？](https://javaguide.cn/database/redis/redis-questions-01.html#redis-内存淘汰策略了解么)

MySQL 里有 2000w 数据，Redis 中只存 20w 的数据，如何保证 Redis 中的数据都是热点数据?

1. **volatile-lru（least recently used）**：从已设置过期时间的数据集（`server.db[i].expires`）中挑选最近最少使用的数据淘汰。
2. **volatile-ttl**：从已设置过期时间的数据集（`server.db[i].expires`）中挑选将要过期的数据淘汰。
3. **volatile-random**：从已设置过期时间的数据集（`server.db[i].expires`）中任意选择数据淘汰。
4. **allkeys-lru（least recently used）**：从数据集（`server.db[i].dict`）中移除最近最少使用的数据淘汰。
5. **allkeys-random**：从数据集（`server.db[i].dict`）中任意选择数据淘汰。
6. **no-eviction**（默认内存淘汰策略）：禁止驱逐数据，当内存不足以容纳新写入数据时，新写入操作会报错。

# Redis性能优化

## [1. 使用批量操作减少网络传输](https://javaguide.cn/database/redis/redis-questions-02.html#使用批量操作减少网络传输)

1. 发送命令
2. 命令排队
3. 命令执行
4. 返回结果

其中，第 1 步和第 4 步耗费时间之和称为 **Round Trip Time (RTT,往返时间)** ，也就是数据在网络上传输的时间。

使用批量操作可以减少网络传输次数，进而有效减小网络开销，大幅减少 RTT。

## [2. Redis bigkey（大 Key）](https://javaguide.cn/database/redis/redis-questions-02.html#redis-bigkey-大-key)

如果一个 key 对应的 value 所占用的内存比较大，那这个 key 就可以看作是 bigkey。

具体多大才算大呢？有一个不是特别精确的参考标准：

- String 类型的 value 超过 1MB
- 复合类型（List、Hash、Set、Sorted Set 等）的 value 包含的元素超过 5000 个（不过，对于复合类型的 value 来说，不一定包含的元素越多，占用的内存就越多）。

**原因：**

- 程序设计不当，比如直接使用 String 类型存储较大的文件对应的二进制数据。
- 对于业务的数据规模考虑不周到，比如使用集合类型的时候没有考虑到数据量的快速增长。
- 未及时清理垃圾数据，比如哈希中冗余了大量的无用键值对。

**危害：**

1. 客户端超时阻塞：由于 Redis 执行命令是单线程处理，然后在操作大 key 时会比较耗时，那么就会阻塞 Redis，从客户端这一视角看，就是很久很久都没有响应。
2. 网络阻塞：每次获取大 key 产生的网络流量较大，如果一个 key 的大小是 1 MB，每秒访问量为 1000，那么每秒会产生 1000MB 的流量，这对于普通千兆网卡的服务器来说是灾难性的。
3. 工作线程阻塞：如果使用 del 删除大 key 时，会阻塞工作线程，这样就没办法处理后续的命令。

[如何发现 bigkey？](https://javaguide.cn/database/redis/redis-questions-02.html#如何发现-bigkey)

**1、使用 Redis 自带的 `--bigkeys` 参数来查找。**

**2、使用 Redis 自带的 SCAN 命令**

**4、借助公有云的 Redis 分析服务。**

## [3. Redis hotkey（热 Key）](https://javaguide.cn/database/redis/redis-questions-02.html#redis-hotkey-热-key)

如果一个 key 的访问次数比较多且明显多于其他 key 的话，那这个 key 就可以看作是 **hotkey（热 Key）**。

处理 hotkey 会占用大量的 CPU 和带宽，可能会影响 Redis 实例对其他请求的正常处理。

[如何发现 hotkey？](https://javaguide.cn/database/redis/redis-questions-02.html#如何发现-hotkey)

**使用 Redis 自带的 `--hotkeys` 参数来查找。**

**根据业务情况提前预估。**

**借助公有云的 Redis 分析服务。**

[如何解决 hotkey？](https://javaguide.cn/database/redis/redis-questions-02.html#如何解决-hotkey)

- **读写分离**：主节点处理写请求，从节点处理读请求。
- **使用 Redis Cluster**：将热点数据分散存储在多个 Redis 节点上。
- **二级缓存**：hotkey 采用二级缓存的方式进行处理，将 hotkey 存放一份到 JVM 本地内存中（可以用 Caffeine）。

## [4. 慢查询命令](https://javaguide.cn/database/redis/redis-questions-02.html#慢查询命令)

[为什么会有慢查询命令？](https://javaguide.cn/database/redis/redis-questions-02.html#为什么会有慢查询命令)

Redis 中的大部分命令都是 O(1)时间复杂度，但也有少部分 O(n) 时间复杂度的命令，例如：

- `KEYS *`：会返回所有符合规则的 key。
- `HGETALL`：会返回一个 Hash 中所有的键值对。
- `LRANGE`：会返回 List 中指定范围内的元素。
- `SMEMBERS`：返回 Set 中的所有元素。
- `SINTER`/`SUNION`/`SDIFF`：计算多个 Set 的交集/并集/差集。

除了这些 O(n)时间复杂度的命令可能会导致慢查询之外， 还有一些时间复杂度可能在 O(N) 以上的命令，例如：

- `ZRANGE`/`ZREVRANGE`：返回指定 Sorted Set 中指定排名范围内的所有元素。时间复杂度为 O(log(n)+m)，n 为所有元素的数量， m 为返回的元素数量，当 m 和 n 相当大时，O(n) 的时间复杂度更小。

[如何找到慢查询命令？](https://javaguide.cn/database/redis/redis-questions-02.html#如何找到慢查询命令)

在 `redis.conf` 文件中，我们可以使用 `slowlog-log-slower-than` 参数设置耗时命令的阈值，并使用 `slowlog-max-len` 参数设置耗时命令的最大记录条数。

当 Redis 服务器检测到执行时间超过 `slowlog-log-slower-than`阈值的命令时，就会将该命令记录在慢查询日志(slow log) 中，这点和 MySQL 记录慢查询语句类似。

## [5. Redis 内存碎片](https://javaguide.cn/database/redis/redis-questions-02.html#redis-内存碎片)

举个例子：操作系统为你分配了 32 字节的连续内存空间，而你存储数据实际只需要使用 24 字节内存空间，那这多余出来的 8 字节内存空间如果后续没办法再被分配存储其他数据的话，就可以被称为内存碎片。

**1、Redis 存储数据的时候向操作系统申请的内存空间可能会大于数据实际需要的存储空间。**

**2、频繁修改 Redis 中的数据也会产生内存碎片。**





![内存碎片](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502041417943.png)

[如何清理 Redis 内存碎片？](https://javaguide.cn/database/redis/redis-memory-fragmentation.html#如何清理-redis-内存碎片)

~~~bash
config set activedefrag yes

# 内存碎片占用空间达到 500mb 的时候开始清理
config set active-defrag-ignore-bytes 500mb
# 内存碎片率大于 1.5 的时候开始清理
config set active-defrag-threshold-lower 50
~~~

# 主从架构、哨兵、分片集群

![image-20250204145357129](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502041453168.png)



