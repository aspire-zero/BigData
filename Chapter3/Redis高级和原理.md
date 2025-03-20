# 分布式缓存

单点redis缺点：

- 数据丢失问题：Redis是内存存储，服务重启可能会丢失数据
- 并发能力问题：单节点Redis并发能力虽然不错，但也无法满足如618这样的高并发场景
- 故障恢复问题：如果Redis宕机，则服务不可用，需要一种自动的故障恢复手段
- 存储能力问题：Redis基于内存，单节点能存储的数据量难以满足海量数据需求

![image-20241101144912775](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241101144913.png)

# 1.Redis的持久化

## 1.1RDB持久化

- RDB全称Redis Database Backup file（Redis数据备份文件），也被叫做Redis数据快照。
- 简单来说就是把内存中的**所有数据**都记录到磁盘中，不是只写更新的数据。
- 当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。
- 快照文件称为RDB文件，默认是保存在当前运行目录。

### 1.1.1执行时机

RDB持久化在四种情况下会执行：

- 执行save命令
- 执行bgsave命令
- Redis停机时
- 触发RDB条件时

**1）save命令**

执行下面的命令，可以立即执行一次RDB：

![image-20241101155351304](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241101155352.png)

save命令会导致主进程执行RDB，这个过程中其它所有命令都会被阻塞。只有在数据迁移时可能用到。

**2）bgsave命令**

下面的命令可以异步执行RDB：

![image-20210725144725943](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241101155302.png)

这个命令执行后会开启独立进程完成RDB，主进程可以持续处理用户请求，不受影响。

**3）停机时**

Redis停机时会执行一次save命令，实现RDB持久化。

**4）触发RDB条件**

Redis内部有触发RDB的机制，可以在redis.conf文件中找到，格式如下：

```properties
# 900秒内，如果至少有1个key被修改，则执行bgsave ， 如果是save "" 则表示禁用RDB
save 900 1  
save 300 10  
save 60 10000 
```

RDB的其它配置也可以在redis.conf文件中设置：

```properties
# 是否压缩 ,建议不开启，压缩也会消耗cpu，磁盘的话不值钱
rdbcompression yes
# RDB文件名称
dbfilename dump.rdb  
# 文件保存的路径目录
dir ./ 
```

### 1.1.2.RDB原理

bgsave开始时会fork主进程得到子进程，子进程共享主进程的内存数据。完成fork后读取内存数据并写入 RDB 文件。

fork采用的是copy-on-write技术：

- 当主进程执行读操作时，访问共享内存；
- 当主进程执行写操作时，则会拷贝一份数据，执行写操作，极限情况内存数据会翻倍。

![image-20210725151319695](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102154252.png)

### 1.1.3.小结

RDB方式bgsave的基本流程？

- fork主进程得到一个子进程，共享内存空间
- 子进程读取内存数据并写入新的RDB文件
- 用新RDB文件替换旧的RDB文件

RDB会在什么时候执行？save 60 1000代表什么含义？

- 默认是服务停止时
- 代表60秒内至少执行1000次修改则触发RDB

RDB的缺点？

- RDB执行间隔时间长，**两次RDB之间写入数据有丢失的风险**
- fork子进程、压缩、写出RDB文件都比较耗时

## 1.2.AOF持久化

### 1.2.1.AOF原理

AOF全称为Append Only File（追加文件）。Redis处理的每一个写命令都会记录在AOF文件，可以看做是**命令日志文件**。

![image-20210725151543640](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102162722.png)



### 1.2.2.AOF配置

AOF默认是关闭的，需要修改redis.conf配置文件来开启AOF：

```properties
# 是否开启AOF功能，默认是no
appendonly yes
# AOF文件的名称
appendfilename "appendonly.aof"
```

AOF的命令记录的频率也可以通过redis.conf文件来配：

```properties
# 表示每执行一次写命令，立即记录到AOF文件
appendfsync always 
# 写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件，是默认方案
appendfsync everysec 
# 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
appendfsync no
```

三种策略对比：

![image-20241102162858313](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102162859.png)

### 1.2.3.AOF文件重写

因为是记录命令，**AOF文件会比RDB文件大的多**。而且AOF会记录对同一个key的多次写操作，但**只有最后一次写操作才有意义**。通过执行bgrewriteaof命令，可以让AOF文件执行重写功能，用最少的命令达到相同效果。

![image-20210725151729118](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102163027.png)

Redis也会在触发阈值时自动去重写AOF文件。阈值也可以在redis.conf中配置：

```properties
# AOF文件比上次文件 增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# AOF文件体积最小多大以上才触发重写 
auto-aof-rewrite-min-size 64mb 
```

### 1.2.4.文件校验

AOF 校验机制是 Redis 在启动时对 AOF 文件进行检查，以判断文件是否完整，是否有损坏或者丢失的数据。这个机制的原理其实非常简单，就是通过使用一种叫做 **校验和（checksum）** 的数字来验证 AOF 文件。这个校验和是通过对整个 AOF 文件内容进行 CRC64 算法计算得出的数字。如果文件内容发生了变化，那么校验和也会随之改变。因此，Redis 在启动时会比较计算出的校验和与文件末尾保存的校验和（计算的时候会把最后一行保存校验和的内容给忽略点），从而判断 AOF 文件是否完整。如果发现文件有问题，Redis 就会拒绝启动并提供相应的错误信息。AOF 校验机制十分简单有效，可以提高 Redis 数据的可靠性。

类似地，RDB 文件也有类似的校验机制来保证 RDB 文件的正确性，这里就不重复进行介绍了。

## 1.3.RDB与AOF对比

RDB和AOF各有自己的优缺点，如果对数据安全性要求较高，在实际开发中往往会**结合**两者来使用。

![image-20210725151940515](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102163103.png)

如何选择?

- Redis 保存的数据丢失一些也没什么影响的话，可以选择使用 RDB。
- 不建议单独使用 AOF，因为时不时地创建一个 RDB 快照可以进行数据库备份、更快的重启以及解决 AOF 引擎错误。
- 如果保存的数据要求安全性比较高的话，建议同时开启 RDB 和 AOF 持久化或者开启 RDB 和 AOF 混合持久化。

**Redis 在重新启动时会根据持久化的策略读取 RDB 和 AOF 文件来恢复内存中的数据。**

RDB 文件恢复：

- 如果 Redis 检测到存在 RDB 文件（通常是 `dump.rdb`），它会在启动时**优先加载**这个快照文件。
- 加载 RDB 文件时，Redis 会将快照中的所有数据一次性加载到内存中，使得内存的状态恢复到生成快照时的状态。

AOF 文件恢复：

- 如果设置了 AOF 持久化（通常称为 `appendonly.aof`），Redis 在重启时也会检查 AOF 文件。
- 如果 AOF 文件存在且 `appendfsync` 配置为 `everysec` 或者其他值，Redis 会使用 AOF 文件中的命令逐条重放，来恢复内存中的数据。

**恢复过程**

- 优先顺序
  - Redis 在启动时会先尝试加载 RDB 文件，如果 RDB 文件存在且有效，Redis 会加载它。
  - 如果没有 RDB 文件，或者 RDB 文件加载失败，Redis 将尝试加载 AOF 文件。
- 数据一致性
  - 使用 RDB 文件重启时，可能会丢失自最后一次快照以来的一些数据（例如在最后一次 RDB 持久化之后的写操作）。
  - 使用 AOF 文件时，如果 AOF 文件是完整的，所有的写操作都将被回放，通常能更好地保证数据一致性，但相对 RDB 可能会导致更长的恢复时间。

# 2.Redis主从

## 2.1.搭建主从架构

单节点Redis的并发能力是有上限的，要进一步提高Redis的并发能力，就需要搭建主从集群，实现读写分离。

主节点负责写（也可以读），从节点只能负责读取（**不能写**）。

假设有A、B两个Redis实例，如何让B作为A的slave节点?

在B节点执行命令：`slaveof A的IP A的port`

![image-20210725152037611](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102193732.png)



具体搭建流程参考课前资料《Redis集群.md》：

## 2.2.主从数据同步原理

### 2.2.1.全量同步

主从第一次建立连接时，会执行**全量同步**，将master节点的所有数据都拷贝给slave节点，流程：

![image-20210725152222497](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102200229.png)

这里有一个问题，master如何得知salve是第一次来连接呢？？

有几个概念，可以作为判断依据：

- **Replication Id**：简称replid，是数据集的标记，id一致则说明是同一数据集。**每一个master都有唯一的replid，slave则会继承master节点的replid**
- **offset**：偏移量，随着记录在repl_baklog中的数据增多而逐渐增大。slave完成同步时也会记录当前同步的offset。**如果slave的offset小于master的offset，说明slave数据落后于master，需要更新**。

因此slave做数据同步，必须向master声明自己的replication id 和offset，master才可以判断到底需要同步哪些数据。

因为slave原本也是一个master，有自己的replid和offset，当第一次变成slave，与master建立连接时，发送的replid和offset是自己的replid和offset。

master判断发现slave发送来的replid与自己的不一致，说明这是一个全新的slave，就知道要做全量同步了。

master会将自己的replid和offset都发送给这个slave，slave保存这些信息。以后slave的replid就与master一致了。

因此，**master判断一个节点是否是第一次同步的依据，就是看replid是否一致**。

如图：

![image-20210725152700914](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102200359.png)

完整流程描述：

- slave节点请求增量同步
- master节点判断replid，发现不一致，拒绝增量同步
- master将完整内存数据生成RDB，发送RDB到slave
- slave清空本地数据，加载master的RDB
- master将RDB期间的命令记录在repl_baklog，并持续将log中的命令发送给slave
- slave执行接收到的命令，保持与master之间的同步

### 2.2.2.增量同步

全量同步需要先做RDB，然后将RDB文件通过网络传输个slave，成本太高了。

因此除了第一次做全量同步，其它大多数时候slave与master都是做**增量同步**。

什么是增量同步？就是只更新slave与master存在差异的部分数据。如图：

![image-20210725153201086](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102202607.png)



### 2.2.3.repl_backlog原理

master怎么知道slave与自己的数据差异在哪里呢?

这就要说到全量同步时的**repl_baklog**文件了。

这个文件是一个固定大小的**环形数组**，也就是说**角标到达数组末尾后，会再次从0开始读写**，这样数组头部的数据就会被覆盖。

repl_baklog中会记录Redis处理过的命令日志及offset，包括master当前的offset，和slave已经拷贝到的offset：

**slave与master的offset之间的差异，就是salve需要增量拷贝的数据了。**

- 随着不断有数据写入，master的offset逐渐变大，slave也不断的拷贝，追赶master的offset：
- 此时，如果有新的数据写入，就会覆盖数组中的旧数据。不过，旧的数据只要是绿色的，说明是已经被同步到slave的数据，即便被覆盖了也没什么影响。因为未同步的仅仅是红色部分。

- 但是，如果slave出现网络阻塞，导致master的offset远远超过了slave的offset： 如果master继续写入新数据，其offset就会覆盖旧的数据，直到将slave现在的offset也覆盖：
- 棕色框中的红色部分，就是尚未同步，但是却已经被覆盖的数据。此时如果slave恢复，需要同步，却发现自己的offset都没有了，无法完成增量同步了。只能做**全量同步**。

![image-20241102202652329](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102202719.png)

## 2.3.主从同步优化

主从同步可以保证主从数据的一致性，非常重要。

可以从以下几个方面来优化Redis主从就集群：

- 在master中配置repl-diskless-sync yes启用无磁盘复制，不写磁盘，直接网络传，这样要求网速要快。
- Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO
- 适当提高repl_baklog的大小，发现slave宕机时尽快实现故障恢复，尽可能避免全量同步
- 限制一个master上的slave节点数量，如果实在是太多slave，则可以采用**主-从-从链式结构**，减少master压力

主从从架构图：

![image-20210725154405899](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102202952.png)

## 2.4.小结

简述全量同步和增量同步区别？

- 全量同步：master将完整内存数据生成RDB，发送RDB到slave。后续命令则记录在repl_baklog，逐个发送给slave。
- 增量同步：slave提交自己的offset到master，master获取repl_baklog中从offset之后的命令给slave

什么时候执行全量同步？

- slave节点第一次连接master节点时
- slave节点断开时间太久，repl_baklog中的offset已经被覆盖时

什么时候执行增量同步？

- slave节点断开又恢复，并且在repl_baklog中能找到offset时

# 3.Redis哨兵

Redis提供了哨兵（Sentinel）机制来实现主从集群的自动故障恢复。

## 3.1.哨兵原理

### 3.1.1.集群结构和作用

哨兵的结构如图：

![image-20210725154528072](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241104200902.png)

哨兵的作用如下：

- **监控**：Sentinel 会不断检查您的master和slave是否按预期工作
- **自动故障恢复**：如果master故障，Sentinel会将一个slave提升为master。当故障实例恢复后也以新的master为主
- **通知**：Sentinel充当Redis客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给Redis的客户端

### 3.1.2.集群监控原理

Sentinel基于**心跳机制**监测服务状态，每隔1秒向集群的每个实例发送ping命令：

•主观下线：如果某sentinel节点发现某实例未在规定时间响应，则认为该实例**主观下线**。

•客观下线：若超过指定数量（quorum）的sentinel都认为该实例主观下线，则该实例**客观下线**。quorum值最好超过Sentinel实例数量的一半。

![image-20210725154632354](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241104200911.png)

### 3.1.3.集群故障恢复原理

一旦发现master故障，sentinel需要在salve中选择一个作为新的master，选择依据是这样的：

- 首先会判断slave节点与master节点断开时间长短，如果超过指定值（down-after-milliseconds * 10）则会排除该slave节点
- 然后判断slave节点的slave-priority值，越小优先级越高，如果是0则永不参与选举
- 如果slave-prority一样，则判断slave节点的offset值，越大说明数据越新，优先级越高
- 最后是判断slave节点的运行id大小，越小优先级越高。

当选出一个新的master后，该如何实现切换呢？

流程如下：

- sentinel给备选的slave1节点发送slaveof no one命令，**让该节点成为master**
- sentinel给所有其它slave发送slaveof 192.168.150.101 7002 命令，**让这些slave成为新master的从节点**，开始从新的master上同步数据。
- 最后，sentinel将故障节点标记为slave，当**故障节点恢复后会自动成为新的master的slave节点**



![image-20210725154816841](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241104200915.png)

### 3.1.4.小结

Sentinel的三个作用是什么？

- 监控
- 故障转移
- 通知

Sentinel如何判断一个redis实例是否健康？

- 每隔1秒发送一次ping命令，如果超过一定时间没有相向则认为是主观下线
- 如果大多数sentinel都认为实例主观下线，则判定服务下线

故障转移步骤有哪些？

- 首先选定一个slave作为新的master，执行slaveof no one
- 然后让所有节点都执行slaveof 新master
- 修改故障节点配置，添加slaveof 新master

## 3.2.搭建哨兵集群

具体搭建流程参考课前资料《Redis集群.md》：

也和redis一样，需要启动，注意**仅需要连接主节点就行**，因为主节点制动从节点信息，**并且给主节点起个名字**，比如`mymaster`，后序配置redisTemplate需要用到。

## 3.3.RedisTemplate

​	在Sentinel集群监管下的Redis主从集群，其节点会因为自动故障转移而发生变化，Redis的客户端必须感知这种变化，及时更新连接信息。Spring的RedisTemplate底层利用lettuce实现了节点的感知和自动切换。

下面，我们通过一个测试来实现RedisTemplate集成哨兵机制。

### 3.3.1.导入Demo工程

首先，我们引入课前资料提供的Demo工程：

### 3.3.2.引入依赖

在项目的pom文件中引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 3.3.3.配置Redis地址

然后在配置文件application.yml中**指定redis的sentinel相关信息**：

因为主节点是动态变化的，我们需要哨兵告诉我主节点和从节点，完成读写分离。

```java
spring:
  redis:
    sentinel:
      master: mymaster
      nodes:
        - 192.168.150.101:27001
        - 192.168.150.101:27002
        - 192.168.150.101:27003
```

### 3.3.4.配置读写分离

在项目的启动类中，添加一个新的bean：

```java
@Bean
public LettuceClientConfigurationBuilderCustomizer clientConfigurationBuilderCustomizer(){
    return clientConfigurationBuilder -> clientConfigurationBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
}
```

这个bean中配置的就是读写策略，包括四种：

- MASTER：从主节点读取
- MASTER_PREFERRED：优先从master节点读取，master不可用才读取replica
- REPLICA：从slave（replica）节点读取
- REPLICA _PREFERRED：优先从slave（replica）节点读取，所有的slave都不可用才读取master

# 4.Redis分片集群

## 4.1.搭建分片集群

主从和哨兵可以解决高可用、高并发读的问题。但是依然有两个问题没有解决：

- 海量数据存储问题

- 高并发写的问题

使用分片集群可以解决上述问题，如图:

![image-20210725155747294](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331638.png)



分片集群特征：

- 集群中有多个master，每个master保存不同数据

- 每个master都可以有多个slave节点

- master之间通过ping监测彼此健康状态

- 客户端请求可以访问集群任意节点，最终都会被转发到正确节点

具体搭建流程参考课前资料《Redis集群.md》

## 4.2.散列插槽

### 4.2.1.插槽原理

Redis会把每一个master节点映射到0~16383共16384个插槽（hash slot）上，查看集群信息时就能看到：

![image-20210725155820320](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331660.png)



数据key不是与节点绑定，而是与插槽绑定。redis会根据key的有效部分计算插槽值，分两种情况：

- key中包含"{}"，且“{}”中至少包含1个字符，“{}”中的部分是有效部分
- key中不包含“{}”，整个key都是有效部分

例如：key是num，那么就根据num计算，如果是{itcast}num，则根据itcast计算。计算方式是利用CRC16算法得到一个hash值，然后对16384取余，得到的结果就是slot值。

![image-20210725155850200](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331666.png) 

如图，在7001这个节点执行set a 1时，对a做hash运算，对16384取余，得到的结果是15495，因此要存储到103节点。

到了7003后，执行`get num`时，对num做hash运算，对16384取余，得到的结果是2765，因此需要切换到7001节点

### 4.2.1.小结

Redis如何判断某个key应该在哪个实例？

- 将16384个插槽分配到不同的实例
- 根据key的有效部分计算哈希值，对16384取余
- 余数作为插槽，寻找插槽所在实例即可

如何将同一类数据固定的保存在同一个Redis实例？

- 这一类数据使用相同的有效部分，例如key都以{typeId}为前缀

## 4.3.集群伸缩

redis-cli --cluster提供了很多操作集群的命令，可以通过下面方式查看：

![image-20210725160138290](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331655.png)

比如，添加节点的命令：

![image-20210725160448139](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331669.png)



### 4.3.1.需求分析

需求：向集群中添加一个新的master节点，并向其中存储 num = 10

- 启动一个新的redis实例，端口为7004
- 添加7004到之前的集群，并作为一个master节点
- 给7004节点分配插槽，使得num这个key可以存储到7004实例

这里需要两个新的功能：

- 添加一个节点到集群中
- 将部分插槽分配到新插槽

### 4.3.2.创建新的redis实例

创建一个文件夹：

```sh
mkdir 7004
```

拷贝配置文件：

```sh
cp redis.conf /7004
```

修改配置文件：

```sh
sed /s/6379/7004/g 7004/redis.conf
```

启动

```sh
redis-server 7004/redis.conf
```

### 4.3.3.添加新节点到redis

添加节点的语法如下：

![image-20210725160448139](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331669.png)

执行命令：

```sh
redis-cli --cluster add-node  192.168.150.101:7004 192.168.150.101:7001
```

通过命令查看集群状态：

```sh
redis-cli -p 7001 cluster nodes
```

如图，7004加入了集群，并且默认是一个master节点：

![image-20210725161007099](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331904.png)

但是，可以看到7004节点的插槽数量为0，因此没有任何数据可以存储到7004上

### 4.3.4.转移插槽

我们要将num存储到7004节点，因此需要先看看num的插槽是多少：

![image-20210725161241793](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331924.png)

如上图所示，num的插槽为2765.

我们可以将0~3000的插槽从7001转移到7004，命令格式如下：

![image-20210725161401925](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331934.png)

具体命令如下：

建立连接：

![image-20210725161506241](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331946.png)

得到下面的反馈：

![image-20210725161540841](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331952.png)

询问要移动多少个插槽，我们计划是3000个：

新的问题来了：

![image-20210725161637152](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331959.png)

那个node来接收这些插槽？？

显然是7004，那么7004节点的id是多少呢？

![image-20210725161731738](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331057.png)

复制这个id，然后拷贝到刚才的控制台后：

![image-20210725161817642](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331065.png)

这里询问，你的插槽是从哪里移动过来的？

- all：代表全部，也就是三个节点各转移一部分
- 具体的id：目标节点的id
- done：没有了

这里我们要从7001获取，因此填写7001的id：

![image-20210725162030478](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331080.png)

填完后，点击done，这样插槽转移就准备好了：

![image-20210725162101228](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331101.png)

确认要转移吗？输入yes：

然后，通过命令查看结果：

![image-20210725162145497](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331116.png) 

可以看到： 

![image-20210725162224058](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331133.png)

目的达成。

## 4.4.故障转移

集群初识状态是这样的：

![image-20210727161152065](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331227.png)

其中7001、7002、7003都是master，我们计划让7002宕机。

### 4.4.1.自动故障转移

当集群中有一个master宕机会发生什么呢？

直接停止一个redis实例，例如7002：

```sh
redis-cli -p 7002 shutdown
```

1）首先是该实例与其它实例失去连接

2）然后是疑似宕机：

![image-20210725162319490](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331248.png)

3）最后是确定下线，自动提升一个slave为新的master：

![image-20210725162408979](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331275.png)

4）当7002再次启动，就会变为一个slave节点了：

![image-20210727160803386](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331288.png)

### 4.4.2.手动故障转移

利用cluster failover命令可以手动让集群中的某个master宕机，切换到执行cluster failover命令的这个slave节点，实现无感知的数据迁移。其流程如下：

![image-20210725162441407](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331358.png)



这种failover命令可以指定三种模式：

- 缺省：默认的流程，如图1~6歩
- force：省略了对offset的一致性校验
- takeover：直接执行第5歩，忽略数据一致性、忽略master状态和其它master的意见

**案例需求**：在7002这个slave节点执行手动故障转移，重新夺回master地位

步骤如下：

1）利用redis-cli连接7002这个节点

2）执行cluster failover命令

如图：

![image-20210727160037766](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331456.png)

效果：

![image-20210727161152065](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202412151331227.png)	

## 4.5.RedisTemplate访问分片集群

RedisTemplate底层同样基于lettuce实现了分片集群的支持，而使用的步骤与哨兵模式基本一致：

1）引入redis的starter依赖

2）配置分片集群地址

3）配置读写分离

与哨兵模式相比，其中只有分片集群的配置方式略有差异，如下：

```yaml
spring:
  redis:
    cluster:
      nodes:
        - 192.168.150.101:7001
        - 192.168.150.101:7002
        - 192.168.150.101:7003
        - 192.168.150.101:8001
        - 192.168.150.101:8002
        - 192.168.150.101:8003
```



