# [MySQL 基础](https://javaguide.cn/database/mysql/mysql-questions-01.html#mysql-基础)

## [1. 什么是关系型数据库？](https://javaguide.cn/database/mysql/mysql-questions-01.html#什么是关系型数据库)

关系型数据库是一种基于关系模型的数据库，使用表格（即“关系”）来存储和管理数据。

关系模型表明了数据库中所存储的数据之间的联系（一对一、一对多、多对多）。

## [2. 什么是 SQL？](https://javaguide.cn/database/mysql/mysql-questions-01.html#什么是-sql)

结构化查询语言(Structured Query Language)，专门用来与数据库打交道，目的是提供一种从数据库中读写数据的简单有效的方法。

## [3.什么是 MySQL？](https://javaguide.cn/database/mysql/mysql-questions-01.html#什么是-mysql)

**MySQL 是一种关系型数据库**，MySQL 的默认端口号是**3306**。

## [4. MySQL 有什么优点？](https://javaguide.cn/database/mysql/mysql-questions-01.html#mysql-有什么优点)

1. 成熟稳定，功能完善。
2. 开源免费。
3. 社区活跃，生态完善。
4. 事务支持优秀， InnoDB 存储引擎默认使用 REPEATABLE-READ 并不会有任何性能损失，并且，InnoDB 实现的 REPEATABLE-READ 隔离级别其实是可以解决幻读问题发生的。
5. 支持分库分表、读写分离、高可用。

# [MySQL 字段类型](https://javaguide.cn/database/mysql/mysql-questions-01.html#mysql-字段类型)

## 1. 有哪些字段？

- **数值类型**：整型（TINYINT、SMALLINT、MEDIUMINT、INT 和 BIGINT）、浮点型（FLOAT 和 DOUBLE）、定点型（DECIMAL）
- **字符串类型**：CHAR、VARCHAR、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT、TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB 等，最常用的是 CHAR 和 VARCHAR。
  - **日期时间类型**：YEAR、TIME、DATE、DATETIME 和 TIMESTAMP 等。

![MySQL 常见字段类型总结](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271140742.png)

## [2. 整数类型的 UNSIGNED 属性有什么用？](https://javaguide.cn/database/mysql/mysql-questions-01.html#整数类型的-unsigned-属性有什么用)

使用 UNSIGNED 属性可以将正整数的上限**提高一倍**，因为它不需要存储负数值。

## [3. CHAR 和 VARCHAR 的区别是什么？](https://javaguide.cn/database/mysql/mysql-questions-01.html#char-和-varchar-的区别是什么)

- CHAR 是定长字符串;
- CHAR 在存储时会在右边填充空格以达到指定的长度，检索时会去掉空格；
- CHAR 更适合存储长度较短或者长度都差不多的字符串，例如 Bcrypt 算法、MD5 算法加密后的密码、身份证号码。



- VARCHAR 是变长字符串。
- VARCHAR 在存储时需要使用 1 或 2 个额外字节记录字符串的长度，检索时不需要处理。
- VARCHAR 类型适合存储长度不确定或者差异较大的字符串，例如用户昵称、文章标题等。

![image-20250127115508137](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271155162.png)

## [4. VARCHAR(100)和 VARCHAR(10)的区别是什么？](https://javaguide.cn/database/mysql/mysql-questions-01.html#varchar-100-和-varchar-10-的区别是什么)

VARCHAR(100)和 VARCHAR(10)都是变长类型，表示能存储最多 100 个字符和 10 个字符。

二者存储相同的字符串，所占用磁盘的存储空间其实是一样的。

不过，VARCHAR(100) 会消耗更多的内存。在进行排序的时候，VARCHAR(100)是按照 100 这个长度来进行的，也就会消耗更多内存。

## [5. DECIMAL 和 FLOAT/DOUBLE 的区别是什么？](https://javaguide.cn/database/mysql/mysql-questions-01.html#decimal-和-float-double-的区别是什么)

DECIMAL 是定点数，FLOAT/DOUBLE 是浮点数。

DECIMAL 可以存储精确的小数值，FLOAT/DOUBLE 只能存储近似的小数值。

MySQL 的 DECIMAL 类型对应的是 Java 类 `java.math.BigDecimal`

## [6. 为什么不推荐使用 TEXT 和 BLOB？](https://javaguide.cn/database/mysql/mysql-questions-01.html#为什么不推荐使用-text-和-blob)

- 不能有默认值。
- 检索效率较低。
- 不能直接创建索引，需要指定前缀长度。

## [7. DATETIME 和 TIMESTAMP 的区别是什么？](https://javaguide.cn/database/mysql/mysql-questions-01.html#datetime-和-timestamp-的区别是什么)

DATETIME 类型没有时区信息，TIMESTAMP 和时区有关。

TIMESTAMP 只需要使用 4 个字节的存储空间，但是 DATETIME 需要耗费 8 个字节的存储空间。

## [8. NULL 和 '' 的区别是什么？](https://javaguide.cn/database/mysql/mysql-questions-01.html#null-和-的区别是什么)

- `''`的长度是 0，是不占用空间的，而`NULL` 是需要占用空间的。
- `NULL` 会影响聚合函数的结果。例如，`SUM`、`AVG`、`MIN`、`MAX` 等聚合函数会忽略 `NULL` 值。
- 查询 `NULL` 值时，必须使用 `IS NULL` 或 `IS NOT NULLl` 来判断，而不能使用 =、!=、 <、> 之类的比较运算符。而`''`是可以使用这些比较运算符的。
- `NULL` 代表一个不确定的值，需要增加判断。

## [9. Boolean 类型如何表示？](https://javaguide.cn/database/mysql/mysql-questions-01.html#boolean-类型如何表示)

MySQL 中没有专门的布尔类型，而是用 TINYINT(1) 类型来表示布尔值。TINYINT(1) 类型可以存储 0 或 1，分别对应 false 或 true。

# [MySQL 基础架构](https://javaguide.cn/database/mysql/mysql-questions-01.html#mysql-基础架构)

## 1. 结构

- **连接器：** 身份认证和权限相关(登录 MySQL 的时候)。
- **查询缓存：** 执行查询语句的时候，会先查询缓存（MySQL 8.0 版本后移除，因为这个功能不太实用）。
- **分析器：** 没有命中缓存的话，SQL 语句就会经过分析器，分析器说白了就是要先看你的 SQL 语句要干嘛，再检查你的 SQL 语句语法是否正确。
- **优化器：** 按照 MySQL 认为最优的方案去执行。
- **执行器：** 执行语句，然后从存储引擎返回数据。 执行语句之前会先判断是否有权限，如果没有权限的话，就会报错。
- **插件式存储引擎**：主要负责数据的存储和读取，采用的是插件式架构，支持 InnoDB、MyISAM、Memory 等多种存储引擎。InnoDB 是 MySQL 的默认存储引擎，绝大部分场景使用 InnoDB 就是最好的选择。

![img](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271208683.png)



## 2. SQL语句在MySQL中的执行过程

- 查询语句的执行流程如下：权限校验（如果命中缓存）--->查询缓存--->分析器--->优化器--->权限校验--->执行器--->引擎
- 更新语句执行流程如下：分析器---->权限校验---->执行器--->引擎---redo log(prepare 状态)--->binlog--->redo log(commit 状态)

# [MySQL 存储引擎](https://javaguide.cn/database/mysql/mysql-questions-01.html#mysql-存储引擎)

## [1. MySQL 支持哪些存储引擎？默认使用哪个？](https://javaguide.cn/database/mysql/mysql-questions-01.html#mysql-支持哪些存储引擎-默认使用哪个)

默认是InnoDB。MySQL 5.5.5 之前，MyISAM 是 MySQL 的默认存储引擎。5.5.5 版本之后，InnoDB 是 MySQL 的默认存储引擎。

InnoDB ，支持事务，支持外键，支持行级锁。

![image-20250127121020990](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271210015.png)

## [2. MySQL 存储引擎架构了解吗？](https://javaguide.cn/database/mysql/mysql-questions-01.html#mysql-存储引擎架构了解吗)

MySQL 存储引擎采用的是 **插件式架构** ，支持多种存储引擎，存储引擎是**基于表的，而不是数据库**。

## [3. MyISAM 和 InnoDB 有什么区别？如何选择？](https://javaguide.cn/database/mysql/mysql-questions-01.html#myisam-和-innodb-有什么区别)

- InnoDB 支持行级别的锁粒度，MyISAM 不支持，只支持表级别的锁粒度。
- MyISAM 不提供事务支持。InnoDB 提供事务支持，实现了 SQL 标准定义了四个隔离级别。
- MyISAM 不支持外键，而 InnoDB 支持。
- MyISAM 不支持 MVCC，而 InnoDB 支持。
- 虽然 MyISAM 引擎和 InnoDB 引擎都是使用 B+Tree 作为索引结构，但是两者的实现方式不太一样。
- MyISAM 不支持数据库异常崩溃后的安全恢复，而 InnoDB 支持。
- InnoDB 的性能比 MyISAM 更强大。



读密集的情况下，使用 MyISAM 也是合适的。对于咱们日常开发的业务系统来说，你几乎找不到什么理由使用 MyISAM 了，老老实实用默认的 InnoDB 就可以了！

# [MySQL 索引](https://javaguide.cn/database/mysql/mysql-questions-01.html#mysql-索引)

## [1. 索引介绍](https://javaguide.cn/database/mysql/mysql-index.html#索引介绍)

索引是一种用于快速查询和检索数据的数据结构，其本质可以看成是一种排序好的**数据结构**。

常见的索引结构有: B 树， B+树 和 Hash、红黑树。在 MySQL 中，无论是 Innodb 还是 MyIsam，都使用了 B+树作为索引结构。

## [2. 索引的优缺点](https://javaguide.cn/database/mysql/mysql-index.html#索引的优缺点)

**优点**：

- 使用索引可以大大加快数据的检索速度（大大减少检索的数据量）, 减少 IO 次数，这也是创建索引的最主要的原因。
- 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。

**缺点**：

- 创建索引和维护索引需要耗费许多时间。当对表中的数据进行增删改的时候，如果数据有索引，那么索引也需要动态的修改，会降低 SQL 执行效率。
- 索引需要使用物理文件存储，也会耗费一定空间。

大多数情况下，索引查询都是比全表扫描要快的。但是如果数据库的数据量不大，那么使用索引也不一定能够带来很大提升。

**百万行以上的数据，索引可以带来显著的性能提示。**

## [3. 索引底层数据结构选型](https://javaguide.cn/database/mysql/mysql-index.html#索引底层数据结构选型)

见MySQL进阶篇。

### 3.1 B+和B-树区别？

**多路平衡查找树**

B+Tree，所有的数据都会出现在叶子节点，非叶子节点仅仅起到索引数据作用。

B+Tree，叶子节点形成一个单向链表。

MySQL优化后的形成了双向循环链表。

![image-20250127122643484](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271226515.png)

### 3.2 为什么MySQL选用B+，不是B-、二叉树、哈希？

- 相对于二叉树，层级更少，搜索效率高；
- 对于B-tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低；
- 相对Hash索引，B+tree支持范围匹配及排序操作；

## 4. 索引分类

### 4.1 按照类型

唯一索引的主要目的不是为了加快查询，而是为了不重复。

![image-20250127144825213](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271448249.png)

### 4.2 按照底层存储方式角度

如果存在主键，**主键索引就是聚集索引**。

如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引。

如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引。

![image-20250127145236603](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271452638.png)

**回表查询**： 这种先到二级索引中查找数据，找到主键值，然后再到聚集索引中根据主键值，获取数据的方式，就称之为回表查询。

![image-20250127145359377](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271453406.png)

### 4.3 覆盖索引

如果一个索引包含（或者说覆盖）所有需要查询的字段的值，我们就称之为 **覆盖索引（Covering Index）** 。

**覆盖索引即需要查询的字段正好是索引的字段，那么直接根据该索引，就可以查到数据了，而无需回表查询。**

![image-20250127150306455](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271503490.png)

### 4.4 前缀索引

当字段类型为字符串（varchar，text，longtext等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO， 影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率

![image-20250127150452126](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271504161.png)

### [4.5 联合索引](https://javaguide.cn/database/mysql/mysql-index.html#联合索引)

使用表中的多个字段创建索引，就是 **联合索引**，也叫 **组合索引** 或 **复合索引**。

## 5. 最左前缀法则

如果索引了多列（联合索引），最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。如果跳跃某一列，索引将会部分失效(**后面的字段索引失效**)。

最左匹配原则会一直向右匹配，直到遇到**范围查询（如 >、<）为止**。对于 >=、<=、BETWEEN 以及前缀匹配 LIKE 的范围查询，不会停止匹配

最左前缀法则中指的最左边的列，是指在查询时，联合索引的最左边的字段(即是第一个字段)必须存在，与我们编写SQL时，条**件编写的先后顺序无关。**



![image-20250127150902796](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271509832.png)

## 6. 索引失效

不要在索引列上进行运算操作， 索引将失效。

![image-20250127151330186](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271513219.png)

字符串不加引号：如果字符串不加单引号，对于查询结果，没什么影响，但是数据库存在隐式类型转换，索引将失效



模糊查询：在like模糊查询中，在关键字后面加%，索引可以生效。而如果在关键字前面加了%，索引将会失效。

![image-20250127151456932](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271514967.png)

or连接条件：如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。

​	

如果MySQL评估使用索引比全表更慢，则不使用索引。

## [7. 正确使用索引的一些建议](https://javaguide.cn/database/mysql/mysql-index.html#正确使用索引的一些建议)

- **不为 NULL 的字段**：索引字段的数据应该尽量不为 NULL，因为对于数据为 NULL 的字段，数据库较难优化。如果字段频繁被查询，但又避免不了为 NULL，建议使用 0,1,true,false 这样语义较为清晰的短值或短字符作为替代。
- **被频繁查询的字段**：我们创建索引的字段应该是查询操作非常频繁的字段。
- **被作为条件查询的字段**：被作为 WHERE 条件查询的字段，应该被考虑建立索引。
- **频繁需要排序的字段**：索引已经排序，这样查询可以利用索引的排序，加快排序查询时间。
- **选择区分度高的列**作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
- **尽量使用联合索引**，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
- **被频繁更新的字段应该慎重建立索引**
- **限制每张表上的索引数量**
- **被经常频繁用于连接的字段**：经常用于连接的字段可能是一些外键列，对于外键列并不一定要建立外键，只是说该列涉及到表与表的关系。对于频繁被连接查询的字段，可以考虑建立索引，提高多表连接查询的效率。

# 日志

## [1. redo log](https://javaguide.cn/database/mysql/mysql-logs.html#redo-log)

重做日志缓冲（redo log buffer）以及重做日志文件（redo logfile）,前者是在内存中，后者在磁盘中。

当事务提交之后会把所有修改信息都存到该**日志文件**中, 用于在**刷新脏页到磁盘,发生错误时, 进行数据恢复使用**。

redo log（重做日志）让 InnoDB 存储引擎拥有了崩溃恢复能力。

我们知道，在InnoDB引擎中的内存结构中，主要的内存区域就是缓冲池，在缓冲池中缓存了很多的数据页。 当我们在一个事务中，执行多个增删改的操作时，InnoDB引擎会先操作缓冲池中的数据，如果缓冲区没有对应的数据，会通过后台线程将磁盘中的数据加载出来，存放在缓冲区中，然后将缓冲池中的数据修改，修改后的数据页我们称为脏页。 而脏页则会在一定的时机，通过后台线程刷新到磁盘

中，从而保证缓冲区与磁盘的数据一致。 而缓冲区的脏页数据并不是实时刷新的，而是一段时间之后将缓冲区的数据刷新到磁盘中，假如刷新到磁盘的过程出错了，而提示给用户事务提交成功，而数据却没有持久化下来，这就出现问题了，没有保证事务的持久性。

![image-20250127194749333](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271947403.png)

有了redolog之后，当对缓冲区的数据进行增删改之后，会首先将操作的数据页的变化，记录在redolog buffer中。

在事务提交时，会将redo log buffer中的数据刷新到redo log磁盘文件中。

过一段时间之后，如果刷新缓冲区的脏页到磁盘时，发生错误，此时就可以借助于redo log进行数据恢复，这样就保证了事务的持久性。 而如果脏页成功刷新到磁盘 或 或者涉及到的数据已经落盘，此时redolog就没有作用了，就可以删除了，所以存在的两个redolog文件是循环写的。

![image-20250127194805086](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271948156.png)

那为什么每一次提交事务，**要刷新redo log 到磁盘中呢**，而不是直接将buffer pool中的脏页刷新到磁盘呢 ?

我们操作数据一般都是随机读写磁盘的，而不是顺序读写磁盘。 而redo log在往磁盘文件中写入数据，由于是日志文件，所以都是顺序写的。顺序写的效率，要远大于随机写。

## [2. undo log](https://javaguide.cn/database/mysql/mysql-logs.html#undo-log)

回滚日志，用于记录数据被修改前的信息 , 作用包含两个 : 提供回滚(**保证事务的原子性**) 和MVCC(多版本并发控制) 。

undo log和redo log记录物理日志不一样，它是**逻辑日志**。可以认为当delete一条记录时，undolog中会记录一条对应的insert记录，反之亦然，当update一条记录时，它记录一条对应相反的update记录。当执行rollback时，就可以从undo log中的逻辑记录读取到相应的内容并进行回滚。

## [3. binlog](https://javaguide.cn/database/mysql/mysql-logs.html#binlog)

## [4. 总结](https://javaguide.cn/database/mysql/mysql-logs.html#总结)

InnoDB 引擎使用 **redo log(重做日志)** 保证事务的**持久性**。

**undo log(回滚日志)** 来保证事务的**原子性**。

锁和MVCC保证了隔离性。

# MVCC

## 1. 当前读、快照读、MVCC

**当前读**：读取的是记录的**最新版本**，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行**加锁**。对于我们日常的操作，如：`select ... lock in share mode(共享锁)`，`select ...for update、update、insert、delete(排他锁`)都是一种当前读。

**快照读**：简单的select（不加锁）就是快照读，快照读，读取的是记录数据的**可见版本，有可能是历史数据**，**不加锁**，是非阻塞读。

![image-20250127201744987](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501272017051.png)

在测试中,我们看到即使事务B提交了数据,事务A中也查询不到。 原因就是因为普通的select是快照读，而在当前默认的RR隔离级别下，开启事务后第一个select语句才是快照读的地方，后面执行相同的select语句都是从快照中获取数据，可能不是当前的最新数据，这样也就保证了可重复读。

**MVCC**：Multi-Version Concurrency Control，多版本并发控制指维护一个数据的多个版本，使得读写操作没有冲突。

依赖于：

- 数据库记录中的三个隐式字段
- undo log日志；
- readView。

## 2. 三个隐藏字段

![image-20250127202154136](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501272021199.png)

## 3. undoLog

回滚日志，在insert、update、delete的时候产生的便于数据回滚的日志。

当insert的时候，产生的undo log日志只在回滚时需要，在事务提交后，可被立即删除。

而update、delete的时候，产生的undo log日志不仅在回滚时需要，在快照读时也需要，不会立即被删除。

![image-20250127202536171](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501272025224.png)

## 4. readview

ReadView（读视图）是 快照读 SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务（未提交的）id。

![image-20250127202806786](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501272028847.png)

![image-20250127202920724](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501272029798.png)

## [5. RC 和 RR 隔离级别下 MVCC 的差异](https://javaguide.cn/database/mysql/innodb-implementation-of-mvcc.html#rc-和-rr-隔离级别下-mvcc-的差异)

- 在 RC（读已提交） 隔离级别下的 **`每次select`** 查询前都生成一个`Read View` (m_ids 列表)
- 在 RR （可重复读）隔离级别下只在事务开始后 **`第一次select`** 数据前生成一个`Read View`（m_ids 列表）

在 RC 隔离级别下，事务在每次查询开始时都会生成并设置新的 Read View，所以导致不可重复读。

RR 隔离级别只会在事务开启后的第一次查询生成 `Read View` ，并使用至事务提交。所以在生成 `Read View` 之后其它事务所做的更新、插入记录版本对当前事务并不可见，实现了可重复读和防止快照读下的 “幻读”。

## [6. MVCC➕Next-key-Lock 防止幻读](https://javaguide.cn/database/mysql/innodb-implementation-of-mvcc.html#mvcc➕next-key-lock-防止幻读)

mysql的RR是可以防止幻读的，具体分为是快照读还是当前读。

**1、执行普通 `select`，此时会以 `MVCC` 快照读的方式读取数据**

在快照读的情况下，RR 隔离级别只会在事务开启后的第一次查询生成 `Read View` ，并使用至事务提交。所以在生成 `Read View` 之后其它事务所做的更新、插入记录版本对当前事务并不可见，实现了可重复读和防止快照读下的 “幻读”

**2、执行 select...for update/lock in share mode、insert、update、delete 等当前读**

在当前读下，读取的都是最新的数据，如果其它事务有插入新的记录，并且刚好在当前事务查询范围内，就会产生幻读！`InnoDB` 使用 [Next-key Lock](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-next-key-locks) 来防止这种情况。**当执行当前读时，会锁定读取到的记录的同时，锁定它们的间隙，防止其它事务在查询范围内插入数据。只要我不让你插入，就不会发生幻读**

# 事务

## [1. 何谓事务？ACID四个特性？](https://javaguide.cn/database/mysql/mysql-questions-01.html#何谓事务)

事务是逻辑上的一组操作，要么都执行，要么都不执行。

![image-20250127164345473](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271643510.png)

- 原子性（Atomicity）：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。
- 一致性（Consistency）：事务完成时，数据都保持一致。例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；
- 隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。
- 持久性（Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的，即使数据库发生故障也不应该对其有任何影响。

**只有保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障，也就是说 A、I、D 是手段，C 是目的！** 

## 2. 四个特性有由什么来保持？

![image-20250127195928661](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271959698.png)

## [3. 并发事务带来了哪些问题?](https://javaguide.cn/database/mysql/mysql-questions-01.html#并发事务带来了哪些问题)

**脏读**：一个事务读取了另一个未提交事务的中间数据，如果未提交事务回滚，读取的数据将无效，导致不一致。

**不可重复读**：同一事务内多次读取同一数据，结果不一致，原因是：其他事务在读取间隙修改了数据并提交。

**幻读**：同一事务内多次查询，结果集不一致（多了或少了数据），他事务在查询间隙插入或删除了数据。

幻读其实可以看作是不可重复读的一种特殊情况。

## [4. 并发事务的控制方式有哪些？](https://javaguide.cn/database/mysql/mysql-questions-01.html#并发事务的控制方式有哪些)

MySQL 中并发事务的控制方式无非就两种：**锁** 和 **MVCC**。

锁可以看作是悲观控制的模式，多版本并发控制（MVCC，Multiversion concurrency control）可以看作是乐观控制的模式。

锁：

- **共享锁（S 锁）**：又称读锁，事务在读取记录的时候获取共享锁，允许多个事务同时获取（锁兼容）。
- **排他锁（X 锁）**：又称写锁/独占锁，事务在修改记录的时候获取排他锁，不允许多个事务同时获取。如果一个记录已经被加了排他锁，那其他事务不能再对这条记录加任何类型的锁（锁不兼容）。

MVCC：

**MVCC** 是多版本并发控制方法，即对一份数据会存储多个版本，通过事务的可见性来保证事务能看到自己应该看到的版本。通常会有一个全局的版本分配器来为每一行数据设置版本号，版本号是唯一的。

## [5. SQL 标准定义了哪些事务隔离级别?](https://javaguide.cn/database/mysql/mysql-questions-01.html#sql-标准定义了哪些事务隔离级别)

- **READ-UNCOMMITTED(读取未提交)** ：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
- **READ-COMMITTED(读取已提交)** ：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
- **REPEATABLE-READ(可重复读)** ：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
- **SERIALIZABLE(可串行化)** ：最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。



![](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271704770.png)

默认的是可重复读。



从上面对 SQL 标准定义了四个隔离级别的介绍可以看出，标准的 SQL 隔离级别定义里，REPEATABLE-READ(可重复读)是不可以防止幻读的。但是！**InnoDB 实现的 REPEATABLE-READ 隔离级别其实是可以解决幻读问题发生的**，主要有下面两种情况：

- **快照读**：由 MVCC 机制来保证不出现幻读。
- **当前读**：使用 Next-Key Lock 进行加锁来保证不出现幻读，Next-Key Lock 是行锁（Record Lock）和间隙锁（Gap Lock）的结合，行锁只能锁住已经存在的行，为了避免插入新行，需要依赖间隙锁。

## [6. MySQL 的隔离级别是基于锁实现的吗？](https://javaguide.cn/database/mysql/mysql-questions-01.html#mysql-的隔离级别是基于锁实现的吗)

基于锁和 MVCC 机制共同实现的。

SERIALIZABLE 隔离级别是通过锁来实现的，READ-COMMITTED 和 REPEATABLE-READ 隔离级别是基于 MVCC 实现的。不过， SERIALIZABLE 之外的其他隔离级别可能也需要用到锁机制，就比如 REPEATABLE-READ 在当前读情况下需要使用加锁读来保证不会出现幻读。

# 锁

## [1. 表级锁和行级锁了解吗？有什么区别？](https://javaguide.cn/database/mysql/mysql-questions-01.html#表级锁和行级锁了解吗-有什么区别)

- **表级锁：** MySQL 中锁定粒度最大的一种锁（全局锁除外），是**针对非索引字段加的锁**，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。不过，触发锁冲突的概率最高，**高并发下效率极低**。表级锁和存储引擎无关，**MyISAM 和 InnoDB 引擎都支持表级锁**。
- **行级锁：** MySQL 中锁定粒度最小的一种锁，是 **针对索引字段加的锁** ，只针对当前操作的行记录进行加锁。 行级锁能大大减少数据库操作的冲突。其加锁粒度最小，**并发度高**，但加锁的开销也最大，加锁慢，会出现死锁。行级锁和存储引擎有关，是在存储引擎层面实现的

## [2. 行级锁的使用有什么注意事项？](https://javaguide.cn/database/mysql/mysql-questions-01.html#行级锁的使用有什么注意事项)

在 InnoDB 默认的**隔离级别 REPEATABLE-READ** 下，行锁默认使用的是 **Next-Key Lock**。

**行级锁只会对查询用到的索引加锁**，如果查询未使用索引或者索引失效的话（where中），MySQL **会退化为表锁**。

如果你查询的条件中用到了 **唯一索引或主键索引**，那么 MySQL 会精准地锁定符合条件的行，使用**记录锁**，特别的如果是给**不存在的记录加锁**（where条件中写：=不存在的数据），使用**间隙锁**。

如果使用的是 **普通索引** 或 **非唯一索引**，则可能锁定范围更大，甚至包括多个不相关的行，使用临键锁。

## [3. InnoDB 有哪几类行锁？](https://javaguide.cn/database/mysql/mysql-questions-01.html#innodb-有哪几类行锁)

在 InnoDB 默认的**隔离级别 REPEATABLE-READ** 下，行锁默认使用的是 **Next-Key Lock**。

- **记录锁（Record Lock）**：属于单个行记录上的锁。
- **间隙锁（Gap Lock）**：锁定一个范围，不包括记录本身，可以防止在这个间隙插入数据，产生幻读。
- **临键锁（Next-Key Lock）**：Record Lock+Gap Lock，锁定一个范围，包含记录本身，主要目的是为了解决幻读问题（MySQL 事务部分提到过）

![image-20250127153218523](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271532563.png)

## [4. 共享锁和排他锁呢？](https://javaguide.cn/database/mysql/mysql-questions-01.html#共享锁和排他锁呢)

不论是**表级锁还是行级锁**，都存在共享锁（Share Lock，S 锁）和排他锁（Exclusive Lock，X 锁）这两类：

- **共享锁（S 锁）**：又称**读锁**，事务在读取记录的时候获取共享锁，允许多个事务同时获取（锁兼容）。
- **排他锁（X 锁）**：又称写锁/独占锁，事务在修改记录的时候获取排他锁，不允许多个事务同时获取。如果一个记录已经被加了排他锁，那其他事务不能再对这条事务加任何类型的锁（锁不兼容）。

**排他锁与任何的锁都不兼容，共享锁仅和共享锁兼容。**

![image-20250127153916343](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271539394.png)

![image-20250127154007038](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271540080.png)

## [5. 意向锁有什么作用？](https://javaguide.cn/database/mysql/mysql-questions-01.html#意向锁有什么作用)

如果需要用到表锁的话，如何判断表中的记录没有行锁呢，一行一行遍历肯定是不行，性能太差。我们需要用到一个叫做意向锁的东东来快速判断是否可以对某个表使用表锁。

![image-20250127155151647](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271551699.png)

**意向锁是由数据引擎自己维护的**，用户无法手动操作意向锁，在执行DML操作时，会对涉及的行加行锁，同时也会对该表加上意向锁。而其他客户端，在对这张表加表锁的时候，会根据该表上所加的意向锁来判定是否可以成功加表锁，而不用逐行判断行锁情况了

![image-20250127155034368](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501271550407.png)

## [6. 当前读和快照读有什么区别？](https://javaguide.cn/database/mysql/mysql-questions-01.html#当前读和快照读有什么区别)

**快照读**（一致性非锁定读）就是单纯的 `SELECT` 语句，但不包括下面这两类 `SELECT` 语句：

~~~mysql
SELECT ... FOR UPDATE
# 共享锁 可以在 MySQL 5.7 和 MySQL 8.0 中使用
SELECT ... LOCK IN SHARE MODE;
# 共享锁 可以在 MySQL 8.0 中使用
SELECT ... FOR SHARE;
~~~

快照读的情况下，如果读取的记录正在执行 UPDATE/DELETE 操作，读取操作不会因此去等待记录上 X 锁的释放，而是会去读取行的一个快照（历史版本）。

快照读比较适合对于数据一致性要求不是特别高且追求极致性能的业务场景。

**当前读** （一致性锁定读）就是给行记录加 X 锁或 S 锁。

~~~mysql
# 对读的记录加一个X锁
SELECT...FOR UPDATE
# 对读的记录加一个S锁
SELECT...LOCK IN SHARE MODE
# 对读的记录加一个S锁
SELECT...FOR SHARE
# 对修改的记录加一个X锁
INSERT...
UPDATE...
DELETE...
~~~

# 三大日志

## InnoDB

见MySQL高级篇。

redo log 它是**物理日志**，记录内容是“在某个数据页上做了什么修改”，属于 InnoDB 存储引擎。

**刷盘时机：**

1. 事务提交：当事务提交时，log buffer 里的 redo log 会被刷新到磁盘（可以通过`innodb_flush_log_at_trx_commit`参数控制，后文会提到）。
2. log buffer 空间不足时：log buffer 中缓存的 redo log 已经占满了 log buffer 总容量的大约一半左右，就需要把这些日志刷新到磁盘上。
3. 正常关闭服务器：MySQL 关闭的时候，redo log 都会刷入到磁盘里去。
4. InnoDB 存储引擎有一个后台线程，每隔`1` 秒，就会把 `redo log buffer` 中的内容写到文件系统缓存（`page cache`），然后调用 `fsync` 刷盘。

**刷盘策略：**

- **0**：设置为 0 的时候，表示每次事务提交时不进行刷盘操作。这种方式性能最高，但是也最不安全，因为如果 MySQL 挂了或宕机了，可能会丢失最近 1 秒内的事务。
- **1**：默认设置，设置为 1 的时候，表示每次事务提交时都将进行刷盘操作。这种方式性能最低，但是也最安全，因为只要事务提交成功，redo log 记录就一定在磁盘里，不会有任何数据丢失。
- **2**：设置为 2 的时候，表示每次事务提交时都只把 log buffer 里的 redo log 内容写入 page cache（文件系统缓存）。page cache 是专门用来缓存文件的，这里被缓存的文件就是 redo log 文件。这种方式的性能和安全性都介于前两者中间。

刷盘策略`innodb_flush_log_at_trx_commit` 的默认值为 1，设置为 1 的时候才不会丢失任何数据。为了保证事务的持久性，我们必须将其设置为 1。也就是说，**一个没有提交事务的 redo log 记录，也可能会刷盘。**

## [binlog](https://javaguide.cn/database/mysql/mysql-logs.html#binlog)

而 binlog 是**逻辑日志**，记录内容是语句的原始逻辑，类似于“给 ID=2 这一行的 c 字段加 1”，属于`MySQL Server` 层。

不管用什么存储引擎，只要发生了表数据更新，都会产生 binlog 日志。

binlog 会记录所有涉及更新数据的逻辑操作，并且是顺序写。

用于 **主从复制（Replication）** 和 **时间点恢复（Point-in-Time Recovery）**。

## 两阶段提交

redo log（重做日志）让 InnoDB 存储引擎拥有了崩溃恢复能力。

binlog（归档日志）保证了 MySQL 集群架构的数据一致性。



redo log 在事务执行过程中可以不断写入，而 binlog 只有在提交事务时才写入，所以 redo log 与 binlog 的写入时机不一样。

![img](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502112147637.png)

我们以`update`语句为例，假设`id=2`的记录，字段`c`值是`0`，把字段`c`值更新成`1`，`SQL`语句为`update T set c=1 where id=2`。

假设执行过程中写完 redo log 日志后，binlog 日志写期间发生了异常，会出现什么情况呢？

由于 binlog 没写完就异常，这时候 binlog 里面没有对应的修改记录。因此，之后用 binlog 日志恢复数据时，就会少这一次更新，恢复出来的这一行`c`值是`0`，而原库因为 redo log 日志恢复，这一行`c`值是`1`，最终数据不一致。

![img](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502112148956.png)

为了解决两份日志之间的逻辑一致问题，InnoDB 存储引擎使用**两阶段提交**方案。

原理很简单，将 redo log 的写入拆成了两个步骤`prepare`和`commit`，这就是**两阶段提交**。

![img](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502112149267.png)

使用**两阶段提交**后，写入 binlog 时发生异常也不会有影响，因为 MySQL 根据 redo log 日志恢复数据时，发现 redo log 还处于`prepare`阶段，并且没有对应 binlog 日志，就会回滚该事务。

核心逻辑。

![img](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502112150263.png)

## [undo log](https://javaguide.cn/database/mysql/mysql-logs.html#undo-log)

undo log 属于逻辑日志，记录的是 SQL 语句，比如说事务执行一条 DELETE 语句，那 undo log 就会记录一条相对应的 INSERT 语句。

保证了原子性、还有MVCC需要用。
