# Redis基础篇

## 简介

Redis是基于内存的键值数据库（NoSql），全名remote dictionary server，远程词典服务器。

Key是id，Value可以是Json格式、也可以是List

![image-20241013213946637](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241013213947.png)



**特性：**

- 键值（`key-value`）型，value支持多种不同数据结构，功能丰富
- **单线程**，每个命令具备原子性
- 低延迟，速度快（**基于内存**、IO多路复用、良好的编码）。
- 支持数据持久化（内存数据定期写入磁盘）
- 支持主从集群、分片集群（水平扩展）
- 支持多语言客户端

Nosql关联性是靠程序员自己维护的，比如放着嵌套json。

![image-20241014083948309](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241014083949.png)

## 基本类型

 ![image-20241014114747107](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241014114748.png)

## 常用命令

### 通用

|  指令  |                        描述                        |
| :----: | :------------------------------------------------: |
|  KEYS  | 查看符合模板的所有key，不建议在生产环境设备上使用  |
|  DEL   |                 删除一个指定的key                  |
| EXISTS |                  判断key是否存在                   |
| EXPIRE | 给一个key设置有效期，有效期到期时该key会被自动删除 |
|  TTL   |              查看一个KEY的剩余有效期               |

### String

String类型，也就是字符串类型，是Redis中最简单的存储类型，不过根据字符串的格式不同：

- `string`：普通字符串
- `int`：整数类型，可以做自增、自减操作
- `float`：浮点类型，可以做自增、自减操作

不管是哪种格式，底层都是**字节数组**形式存储，只不过是编码方式不同。字符串类型的最大空间不能超过**512m**.

|    命令     |                             描述                             |
| :---------: | :----------------------------------------------------------: |
|     SET     |         添加或者修改已经存在的一个String类型的键值对         |
|     GET     |                 根据key获取String类型的value                 |
|    MSET     |                批量添加多个String类型的键值对                |
|    MGET     |             根据多个key获取多个String类型的value             |
|    INCR     |                     让一个整型的key自增1                     |
|   INCRBY    | 让一个整型的key自增并指定步长，例如：incrby num 2 让num值自增2 |
| INCRBYFLOAT |              让一个浮点类型的数字自增并指定步长              |
|    SETNX    | 添加一个String类型的键值对，前提是这个key不存在，否则不执行  |
|    SETEX    |          添加一个String类型的键值对，并且指定有效期          |

**Key的层级**

例如我们的项目名称叫 `heima`，有`user`和`product`两种不同类型的数据，我们可以这样定义key：

可以把java对象序列化为json存储。缺点：修改某个属性的话，整个字符串都得修改

~~~
set heima:user:1 '{“id”:1, “name”: “Jack”, “age”: 21}'
~~~

可视化软件打开就有了层次结构。

|       KEY       |                   VALUE                   |
| :-------------: | :---------------------------------------: |
|  heima:user:1   |    {“id”:1, “name”: “Jack”, “age”: 21}    |
| heima:product:1 | {“id”:1, “name”: “小米11”, “price”: 4999} |

### Hash

Hash类型，也叫散列，其value是一个无序字典，类似于Java中的`HashMap`结构..

![image-20241014135154650](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241014135155.png)

|         命令         |                             描述                             |
| :------------------: | :----------------------------------------------------------: |
| HSET key field value |              添加或者修改hash类型key的field的值              |
|    HGET key field    |                获取一个hash类型key的field的值                |
|    HDEL key field    |                       移除Hash中的字段                       |
|       DEL key        |                         移除整个Hash                         |
|        HMSET         |       hmset 和 hset 效果相同 ，4.0之后hmset可以弃用了        |
|        HMGET         |              批量获取多个hash类型key的field的值              |
|       HGETALL        |         获取一个hash类型的key中的所有的field和value          |
|        HKEYS         |             获取一个hash类型的key中的所有的field             |
|        HVALS         |             获取一个hash类型的key中的所有的value             |
|       HINCRBY        |           让一个hash类型key的字段值自增并指定步长            |
|        HSETNX        | 添加一个hash类型的key的field值，前提是这个field不存在，否则不执行 |

### List

Redis中的List类型与Java中的LinkedList类似，可以看做是一个**双向链表**结构。既可以支持正向检索和也可以支持反向检索。

- 有序
- 元素可以重复
- 插入和删除快
- 查询速度一般

常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等。

|          命令           |                             描述                             |
| :---------------------: | :----------------------------------------------------------: |
|   LPUSH key element …   |                 向列表左侧插入一个或多个元素                 |
|        LPOP key         |        移除并返回列表左侧的第一个元素，没有则返回nil         |
| **RPUSH key element …** |                 向列表右侧插入一个或多个元素                 |
|        RPOP key         |                移除并返回列表右侧的第一个元素                |
|   LRANGE key star end   |                 返回一段角标范围内的所有元素                 |
|      BLPOP和BRPOP       | 与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil |

如何利用List结构模拟一个阻塞队列?

- 入口和出口在不同边
- 出队时采用BLPOP或BRPOP

### Set

Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。

因为也是一个hash表，因此具备与HashSet类似的特征

- 无序
- 元素不可重复
- 查找快
- 支持交集、并集、差集等功能

|         命令         |            描述             |
| :------------------: | :-------------------------: |
|  SADD key member …   |  向set中添加一个或多个元素  |
|  SREM key member …   |     移除set中的指定元素     |
|      SCARD key       |     返回set中元素的个数     |
| SISMEMBER key member | 判断一个元素是否存在于set中 |
|       SMEMBERS       |     获取set中的所有元素     |
|  SINTER key1 key2 …  |     求key1与key2的交集      |
|  SDIFF key1 key2 …   |     求key1与key2的差集      |
|  SUNION key1 key2 …  |     求key1和key2的并集      |

### SortedSet

Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。

SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。

- 可排序
- 元素不重复
- 查询速度快
- score相同，按字典序排序

因为SortedSet的可排序特性，经常被用来**实现排行榜**这样的功能。

|             命令             |                             描述                             |
| :--------------------------: | :----------------------------------------------------------: |
|    ZADD key score member     | 添加一个或多个元素到sorted set ，如果已经存在则更新其score值 |
|       ZREM key member        |                删除sorted set中的一个指定元素                |
|      ZSCORE key member       |             获取sorted set中的指定元素的score值              |
|       ZRANK key member       |              获取sorted set 中的指定元素的排名               |
|          ZCARD key           |                  获取sorted set中的元素个数                  |
|      ZCOUNT key min max      |           统计score值在给定范围内的所有元素的个数            |
| ZINCRBY key increment member |    让sorted set中的指定元素自增，步长为指定的increment值     |
|      ZRANGE key min max      |          按照score排序后，获取指定排名范围内的元素           |
|  ZRANGEBYSCORE key min max   |          按照score排序后，获取指定score范围内的元素          |
|    ZDIFF、ZINTER、ZUNION     |                      求差集、交集、并集                      |

注意：所有的排名**默认都是升序**，如果要降序则在命令的Z后面添加`REV`即可。

## Java客户端

![image-20241014150537756](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241014150538.png)

### Jedis

以Redis命令作为方法名称，学习成本低，简单实用。

但是Jedis实例是**线程不安全**的，多线程环境下需要基于连接池来使用

~~~java
public class JedisConnectionFactory {
    private static final JedisPool jedisPool;
    static {
        //配置连接池
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        //最多的Jedis
        jedisPoolConfig.setMaxTotal(8);
        //最多存在8个Jedis
        jedisPoolConfig.setMaxIdle(8);
        //最少存在0个Jedis
        jedisPoolConfig.setMinIdle(0);
        //如果jedis满了，1秒后就会报错，默认是-1（一直等待jedis）
        jedisPoolConfig.setMaxWaitMillis(200);
        //创建连接池对象
        jedisPool = new JedisPool(jedisPoolConfig,"localhost",6379,1000,"123");
    }
    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
~~~

### SpringDataRedis

SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做`SpringDataRedis`。

- 提供了对不同Redis客户端的整合（`Lettuce`和`Jedis`）
- 提供了**RedisTemplate统一API**来操作Redis
- 支持Redis的发布订阅模型
- 支持Redis哨兵和Redis集群
- 支持基于Lettuce的响应式编程
- 支持基于JDK、JSON、字符串、Spring对象的**数据序列化及反序列化**
- 支持基于Redis的JDKCollection实现

![image-20241014194838425](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241014194839.png)

RedisTemplate可以接收任意Object作为值写入Redis，只不过写入前会把Object序列化为字节形式，**默认是采用JDK序列化**，得到的结果是这样的

![image-20241015091549737](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241015091551.png)



**缺点：**可读性差，内存占用较大。

**序列化方法1：**

我们可以通过自定义RedisTemplate序列化的方式来解决。

~~~java
public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory factory){
        // 1.创建RedisTemplate对象
        RedisTemplate<String ,Object> redisTemplate = new RedisTemplate<>();
        // 2.设置连接工厂
        redisTemplate.setConnectionFactory(factory);
        // 3.创建序列化对象
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        // 4.设置key和hashKey采用String的序列化方式
        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);
        // 5.设置value和hashValue采用json的序列化方式
        redisTemplate.setValueSerializer(genericJackson2JsonRedisSerializer);
        redisTemplate.setHashValueSerializer(genericJackson2JsonRedisSerializer);
        return redisTemplate;
    }
~~~

此时我们已经将RedisTemplate的key设置为`String序列化`，value设置为`Json序列化`的方式，执行插入对象。

![image-20241015091748004](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241015091749.png)

如上图所示，为了在反序列化时知道对象的类型，JSON序列化器会将类的class类型写入json结果中，存入Redis，会带来额外的内存开销。

**序列化方法2：**

为了节省内存空间，我们并不会使用JSON序列化器来处理value，而是统一使用String序列化器，**StringRedisTemplate**默认使用的就是String序列化器，当然这只能存储String类型的key和value。当需要存储Java对象时，使用**ObjectMappe**手动完成对象的序列化和反序列化。

~~~java
@SpringBootTest
class RedisStringTemplateTest {
	@Resource
	private StringRedisTemplate stringRedisTemplate;
	@Test
	void testSaveUser() throws JsonProcessingException {
		// 1.创建一个Json序列化对象
		ObjectMapper objectMapper = new ObjectMapper();
		// 2.将要存入的对象通过Json序列化对象转换为字符串
		String userJson1 = objectMapper.writeValueAsString(new User("Vz", 21));
		// 3.通过StringRedisTemplate将数据存入redis
		stringRedisTemplate.opsForValue().set("user:100",userJson1);
		// 4.通过key取出value
		String userJson2 = stringRedisTemplate.opsForValue().get("user:100");
		// 5.由于取出的值是String类型的Json字符串，因此我们需要通过Json序列化对象来转换为java对象
		User user = objectMapper.readValue(userJson2, User.class);
		// 6.打印结果
		System.out.println("user = " + user);
	}
}
~~~

# 黑马点评

## Session理解

当用户首次访问Web应用时，服务器会自动创建一个新的session。这通常是通过调用`HttpServletRequest.getSession(true)`来实现的，其中`true`参数表示如果不存在session，则创建一个新的session，session有一个id，id被存储在客户端的cookie中。

如果客户端的cookie没有被删除，当用户再次请求服务器时候，服务器就能根据cookie中的sessionid来在tomcat中获得session。

如果用户长时间没有访问服务器，服务器的session可能会过期，即使此时客户端的cookie没有被删除，也会要求用户重新登录，服务器会为用户分配一个新的Session ID，并将其发送回客户端。客户端（浏览器）会更新Cookie中的Session ID，以便后续的请求使用新的会话标识。这样做是为了确保会话的安全性，防止旧的、可能被泄露的Session ID被滥用。

Session管理和Session ID的创建、发送和更新都由Servlet容器（如Tomcat、Jetty等）自动处理。

**Session共享问题**

![image-20241015160215274](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241015160216.png)

## Redis登录逻辑分析

![image-20241015162346694](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241015162347.png)

可以使用Hash，也可以采用String存取json文件。

前段如何存储token？我们返回给前端token，前端可使用浏览器的SessionStorage存储token，然后给ajox添加一个拦截器，所有从前端发送的请求都在header里面加上一个字段'authorization'存放token。

![image-20241015162320709](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241015162321.png)

用户登录状态？业务要求：只要你访问了页面，就应该刷新redis有效期。

![image-20241016083742733](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241016083743.png)

## 缓存

缓存的优点：

- 降低后端负担
- 提高读写效率，降低响应时间

缓存的缺点：

- 数据一致性成本
- 代码维护成本
- 运维成本

![image-20241016084929879](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241016084931.png)

## 商户查询缓存

![image-20241016090049096](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241016090050.png)

## 缓存更新策略

​	由于我们的缓存数据源来自数据库，而数据库的数据是会发生变化的，因此，如果当数据库中数据发生变化，而缓存却没有同步，此时就会有一致性问题存在：用户使用缓存中的过时数据，从而影响业务，产品口碑等。

![image-20241016160714106](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241016160715.png)

方案三解释：

我们进行了十次更新，都在缓存中执行。第十次更新后，线程把缓存写到数据库中，这样数据库只更新了一次，效率高。

缺点：难以保证数据一致性，如果缓存丢失，线程还没来得及写入数据库，就会发生问题。

![image-20241016161255449](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241016161256.png)

是更新缓存，还是删除缓存？为了避免无效的写缓存（很长一段时间压根没人来查缓存），我们可以直接删除缓存。

更新缓存还会造成线程安全问题。

![image-20241016162928224](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241016162929.png)

是先删除缓存，还是先更新数据库？（要保证线程安全问题）

方案2，需要查询缓存的时候恰好缓存超时过期，且在写入缓存这种微妙级操作中，又来了一个线程去更新数据库和缓存。

![image-20241016164359215](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241016164400.png)



## 缓存穿透

一般采用方案1：缓存空对象。

**缓存空对象缺点：**

我如果乱编不存在的id，redis中会出现大量无效数据，造成额外内存消耗，当然应对措施是给数据设置一个较短的ddl。

还可能出现短期不一致，如果这个数据被插入数据库中了，redis还是会返回null，当然应对措施是插入新数据的时候删除redis。

**布隆过滤缺点：**

可能会存在误判，布隆过滤拒绝的，数据库中一定不存在，布隆过滤允许的，可能数据库中不存在，是个概率拦截模型。

![image-20241017092313920](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241017092315.png)

还有以下措施：

- 增强id的复杂度，避免被猜测id规律
- 做好数据的基础格式校验，校验id是否合法，过短和过长的id直接可以pass掉
- 加强用户权限校验，比如一些页面必须登录才能操作
- 做好热点参数的限流，监测到商品被访问的特别多，可以加限流

查询商铺：

![image-20241017092923749](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241017092924.png)

## 缓存雪崩

- 给不同的Key的TTL添加随机值，让其在不同时间段分批失效
- 利用Redis集群提高服务的可用性（使用一个或者多个哨兵(`Sentinel`)实例组成的系统，对redis节点进行监控，在主节点出现故障的情况下，能将从节点中的一个升级为主节点，进行故障转义，保证系统的可用性。
- 给缓存业务添加降级限流策略（自然灾害导致Redis服务器受损，为了保护数据库，我们可以采取快速失败，避免业务继续往后）
- 业务添加多级缓存（浏览器访问静态资源时，优先读取浏览器本地缓存；访问非静态资源（ajax查询数据）时，访问服务端；请求到达Nginx后，优先读取Nginx本地缓存；如果Nginx本地缓存未命中，则去直接查询Redis（不经过Tomcat）；如果Redis查询未命中，则查询Tomcat；请求进入Tomcat后，优先查询JVM进程缓存；如果JVM进程缓存未命中，则查询数据库）

![image-20241017095413108](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241017095414.png)

## 缓存击穿

**发生情况？**

一件秒杀中的商品的key突然失效了，大家都在疯狂抢购，那么这个瞬间就会有无数的请求访问去直接抵达数据库，从而造成缓存击穿。

如果重建缓存需要**多个表查询**，这样重建过程可能得到几百毫秒，这时候又来无数请求访问，都在重建，数据库可能会宕机。

**与缓存雪崩区别**

缓存雪崩是大量的key同时失效，缓存击穿是热门的key失效。

![image-20241017100051707](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241017100052.png)

**互斥锁：**

​	利用锁的互斥性，假设线程过来，只能一个人线程去访问数据库，重建缓存，其他线程只能休眠一会，在尝试重新查询缓存。从而避免对数据库频繁访问产生过大压力，但这也会**影响查询的性能**，将查询的性能从并行变成了串行。

​	缺点在于有锁的情况，就可能死锁：我在进行业务的时候需要查询多个缓存，但是一个缓存在别的业务里面被加了锁，别的业务又需要你这个业务加的锁，就出现了死锁：无限等待。

**逻辑过期：**

​	我们之所以会出现缓存击穿问题，主要原因是在于我们对key设置了TTL，如果我们不设置TTL，那么就不会有缓存击穿问题，但是不设置TTL，数据又会一直占用我们的内存，所以我们可以采用逻辑过期方案，

​	假设线程1去查询缓存，然后从value中判断当前数据已经过期了，此时线程1去获得互斥锁，那么其他线程会进行阻塞，获得了锁的进程他会**开启一个新线程**去进行之前的重建缓存数据的逻辑，直到新开的线程完成者逻辑之后，才会释放锁，而线程1直接进行返回，假设现在线程3过来访问，由于线程2拿着锁，所以线程3无法获得锁，线程3也直接返回数据（但只能返回旧数据，牺牲了数据一致性，换取性能上的提高），只有等待线程2重建缓存数据之后，其他线程才能返回正确的数据。

​	优点是不会像互斥锁一样，很多线程一直等待，性能好。

​	缺点是在重建完缓存数据之前，返回的都是**脏数据**，而且有额外的内存消耗。

![image-20241017100955463](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241017100959.png)

两种方案：一种保证一致性，一种保证可用性，都可以。

![image-20241017101612320](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241017101613.png)

**互斥锁**

![image-20241017151455767](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241017151456.png)

**逻辑过期**

![image-20241017152812228](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241017152813.png)

## 全局唯一ID

订单表如果使用数据库自增ID就会存在一些问题

- id规律性太明显，那么对于用户或者竞争对手，就很容易猜测出我们的一些敏感信息。
- 受单表数据量的限制, MySQL的单表容量不宜超过500W，数据量过大之后，我们就要进行拆库拆表，多个表之间id会重复。

全局id特性

- 唯一性
- 高可用，不能挂掉
- 高性能，不能耽误时间
- 递增性，随时间递增
- 安全性，不能太有规律



前面32位是时间戳，后面是计数器，计数器可以设定为每天的销售量，存入redis中。

![image-20241018100234453](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241018100236.png)

~~~java
@Component
public class RedisIdWorker {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    //设置起始时间，我这里设定的是2022.01.01 00:00:00，从1970年到现在的秒数。
    public static final Long BEGIN_TIMESTAMP = 1640995200L;
    //序列号长度
    public static final Long COUNT_BIT = 32L;
		//keyprefix是业务名
    public long nextId(String keyPrefix){
        //1. 生成时间戳
        LocalDateTime now = LocalDateTime.now();
        long currentSecond = now.toEpochSecond(ZoneOffset.UTC);
        long timeStamp = currentSecond - BEGIN_TIMESTAMP;
        //2. 生成序列号，以天为单位，每天是不同的key
        String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));
        //不会空指针，如果是新的一天的第一条数据，会自动插入一个1
        long count = stringRedisTemplate.opsForValue().increment("inc:" + keyPrefix + ":" + date);
        //3. 拼接并返回，简单位运算
        return timeStamp << COUNT_BIT | count;
    }
}

~~~

## 秒杀功能

![image-20241018140849688](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241018140851.png)

## 超卖问题

假设现在只剩下一张优惠券，线程1过来查询库存，判断库存数大于1，但还没来得及去扣减库存，此时库线程2也过来查询库存，发现库存数也大于1，那么这两个线程都会进行扣减库存操作，最终相当于是多个线程都进行了扣减库存，那么此时就会出现超卖问题。

超卖问题是典型的多线程安全问题，针对这一问题的常见解决方案就是加锁：而对于加锁，我们通常有两种解决方案

1. 悲观锁
   - 悲观锁认为线程安全问题一定会发生，因此在操作数据之前先获取锁，确保线程**串行执行**
   - 例如Synchronized、Lock等，都是悲观锁
2. 乐观锁
   - 乐观锁认为线程安全问题不一定会发生，因此不加锁，只是在更新数据的时候再去判断有没有其他线程对数据进行了修改
     - 如果没有修改，则认为自己是安全的，自己才可以更新数据
     - 如果已经被其他线程修改，则说明发生了安全问题，此时可以重试或者异常

悲观锁的性能低。

**乐观锁**

**版本号法：**

添加一个版本号，每次修改的时候会对版本号进行加1，修改条件也判断版本号是不是之前的版本号。

实际上利用了mysql的update语句带锁。

![image-20241018144247122](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241018144248.png)



**CAS方法**

Compare-And-Swap

可以进行简化，直接在where 语句后面判断此时的库存是不是之前查询的库存，这和版本号做的一样的事情。

**但是缺点是成功率太低！**

我们可以直接把语句继续优化，库存大于0就行。

但是有些问题只能用相等比较，解决方法是把数据存在多张表，这样每个锁的数据就少，线程可以抢多个锁。

**二者比较**

**悲观锁**：添加同步锁，让线程串行执行

- 优点:简单粗暴
- 缺点:性能一般

**乐观锁**：不加锁，在更新时判断是否有其它线程在修改

- 优点:性能好
- 缺点:存在成功率低的问题

## 一人一单

只能用悲观锁，因为乐观锁是修改，之前有数据才能比较数据有没有变。我们业务是查询，只能用悲观锁。

**代理对象**

在 Spring 中，事务管理是通过代理实现的，因此在获取数据库事务时必须使用代理对象。

- **`this` 是普通对象**：在同步方法中使用 `this` 作为锁，意味着你是在锁住当前对象的实例。如果你在这个实例的方法中调用其他需要事务管理的方法，直接使用 `this` 可能不会触发 AOP 相关的增强，因为 AOP 代理只会在通过代理对象进行的调用时生效。
- **需要代理对象来获取事务功能**：为了利用 AOP 的切面功能（比如事务管理），必须使用 `AopContext.currentProxy()` 来获得当前的代理对象。这样可以确保执行的方法具有同步及其它切面逻辑的 Enhancements。

**使用Synchronized修饰事务方法，同步为什么会失效?**

[spring事务管理中，使用Synchronized修饰事务方法，同步为什么会失效_spring synchronized-CSDN博客](https://blog.csdn.net/weixin_54401017/article/details/129768305)

简述：当A线程执行完保存操作后就会去释放锁，而此时还没有提交事务，B线程获取锁后，通过用户名查询，由于[数据库隔离级别](https://so.csdn.net/so/search?q=数据库隔离级别&spm=1001.2101.3001.7020)，不能查询到未提交的数据，所以B线程进行了二次插入操作，等执行完后它们一起提交事务，就会出现脏写这种线程安全问题了。

**为什么要用intern**

![image-20241018194019011](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241018194020.png)

~~~java
 public Result seckillVoucher(Long voucherId) {
        // 1 查询优惠卷
        SeckillVoucher seckillVoucher = seckillVoucherService.getById(voucherId);
        // 2 判断开始时间
        if(seckillVoucher.getBeginTime().isAfter(LocalDateTime.now())){
            return Result.fail("秒杀未开始！");
        }
        // 3 判断结束时间
        if(seckillVoucher.getEndTime().isBefore(LocalDateTime.now())){
            return Result.fail("秒杀已结束！");
        }
        // 4 判断库存
        if(seckillVoucher.getStock() < 1){
            return Result.fail("库存不足！");
        }
        Long userId = UserHolder.getUser().getId();
        // intern() 方法会检查常量池中是否已有相同字符的字符串。如果有，返回池中的引用；
        // 如果没有，JVM会把这个字符串添加到常量池中，并返回它的引用
        // 悲观锁,锁是用户，这样性能高
        // 先获取锁，在提交事务，在释放锁！！！
        synchronized (userId.toString().intern()) {
            //获取代理对象，事务
            //不能是this，因为this是普通对象的方法，没有事务功能，得拿到代理对象（普通对象的增强）
            IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
            return proxy.createVoucherOrder(voucherId);
        }
    }

    @Transactional
    public Result createVoucherOrder(Long voucherId) {
        // 5 判断是否下过单
        Long userId = UserHolder.getUser().getId();
        long orderId;
        int count = query().eq("voucher_id", voucherId).eq("user_id", userId).count();
        // 6 如果下过单，不能下了
        if(count > 0) {
            return Result.fail("不能重复下单！");
        }
        // 7 扣减库存,乐观锁
        boolean success = seckillVoucherService.update()
                .setSql("stock = stock - 1")
                .eq("voucher_id", voucherId)
                .gt("stock",0)
                .update();
        if(!success){
            return Result.fail("库存不足！");
        }
        // 8 创建订单
        VoucherOrder voucherOrder = new VoucherOrder();
        orderId = redisIdWorker.nextId("order");
        voucherOrder.setId(orderId);
        voucherOrder.setUserId(userId);
        voucherOrder.setVoucherId(voucherId);
        save(voucherOrder);
        // 9 返回订单id
        return Result.ok(orderId);
    }
~~~

## 集群环境下的并发问题

​	我们部署了多个Tomcat，每个Tomcat都有一个属于自己的jvm，那么假设在服务器A的Tomcat内部，有两个线程，即线程1和线程2，如果这两个线程的锁对象是同一个，比如“352”字符串为锁，是可以实现互斥的。

​	但是如果在另一个Tomcat的内部，又有两个线程，但是他们的锁对象虽然写的和服务器A一样是“352”，线程3和线程4可以实现互斥，但是却无法和线程1和线程2互斥。

​	因为**锁机制是在 Java 虚拟机的内部实现的**，而每个 JVM 都有自己的内存空间和对象。如果你在服务器A上将字符串“352”用作锁，那么这个锁只对服务器A的 JVM 内部的线程（如线程1和线程2）有效。线程3和线程4虽然使用的是同样的字符串“352”来作为锁，但这个字符串对象在它们自己的 JVM 中是一个全新的对象，完全与服务器A中的“352”锁无关

## 分布式锁

满足分布式系统或集群模式下**多进程**（JVM）课件并且可以互斥的锁。

应该满足：

- 可见性：多个进程都能看到相同的结果。
- 互斥：互斥是分布式锁的最基本条件，使得程序串行执行
- 高可用：程序不易崩溃，时时刻刻都保证较高的可用性
- 高性能：由于加锁本身就让性能降低，所以对于分布式锁需要他较高的加锁性能和释放锁性能
- 安全性：如果锁没释放，程序崩了，会不会发生死锁。

![image-20241019121818305](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241019121819.png)

**MySQL**：本身就带有锁机制，但是由于MySQL的性能一般，所以采用分布式锁的情况下，使用MySQL作为分布式锁比较少见，但是mysql安全性能较好，即使程序崩了，锁也会释放。

**Redis**：Redis作为分布式锁是非常常见的一种使用方式，现在企业级开发中基本都是用Redis或者Zookeeper作为分布式锁，利用`SETNX`这个方法，如果插入Key成功，则表示获得到了锁，如果有人插入成功，那么其他人就回插入失败，无法获取到锁，利用这套逻辑完成`互斥`，从而实现分布式锁。释放锁删除key就行。

缺点是：如果程序崩了，只能利用锁的超时时间来释放锁。

**Zookeeper**：

1. **创建锁节点**：
   - 客户端在 Zookeeper 中创建一个临时节点（znode），并设置为顺序节点（例如，命名为 `/lock/lock-`）。
   - Zookeeper 会自动为这个节点追加一个序列号，形成一个唯一标识，是递增的。
2. **获取锁的顺序**：
   - 客户端获取所有以 `/lock/` 为前缀的节点并排序。
   - 判断当前客户端的节点是否是最小的序列号节点。如果是，则获得锁；如果不是，则需要等待。
3. **等待锁**：
   - 当客户端创建的节点不是最小节点时，客户端需要监听比自己序列号小的节点的删除事件。
   - 如果有其他客户端释放锁（删除其节点），则会**触发监听**，客户端将重新检查节点顺序，判断自己是否为最小节点。
4. **释放锁**：
   - 客户端使用完共享资源后，删除自己创建的临时节点以释放锁。
   - 这将触发其他等待锁的客户端，可能会让它们获得锁。

|        |           Mysql           |          Redis           |            Zookeeper             |
| :----: | :-----------------------: | :----------------------: | :------------------------------: |
|  互斥  | 利用mysql本身的互斥锁机制 | 利用setnx这样的互斥命令  | 利用节点的唯一性和有序性实现互斥 |
| 高可用 |            好             |            好            |                好                |
| 高性能 |           一般            |            好            |               一般               |
| 安全性 |   断开连接，自动释放锁    | 利用锁超时时间，到期释放 |    临时节点，断开连接自动释放    |

### Redis实现思路

获取锁：互斥、**非阻塞**（尝试一次，成功返回true，失败返回false）

释放锁：手动释放+超时释放（保底）

注意，不能使用setnx命令，因为使用setnx命令，要再使用一个expire设置过期时间，需要保证原子性，否则可能没来得及设置过期时间你的程序就宕机了，产生死锁。

~~~txt
SET lock thread01 NX EX 10 //成功返回ok，失败返回nil，NX是互斥，EX是超时时间
DEL lock
~~~

什么是阻塞式获取锁？

如果锁已经被其他线程占用，该线程将会被挂起，直到锁被释放为止。

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20241019125556179.png" alt="image-20241019125556179"  />

### 实现代码

注：代码不完全版本

 ~~~java
 public class SimpleRedisLock implements ILock{
     private static final String KEY_PREFIX = "lock:";
     private String name;
     //不是spring管理的类，不能自动给你注入
     private StringRedisTemplate stringRedisTemplate;
 
     public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
         this.name = name;
         this.stringRedisTemplate = stringRedisTemplate;
     }
 
     @Override
     public boolean tryLock(long timeoutSec) {
         long threadId = Thread.currentThread().getId();
         Boolean success = stringRedisTemplate
                 .opsForValue().setIfAbsent(KEY_PREFIX + name, threadId + "", timeoutSec, TimeUnit.SECONDS);
         // 避免拆箱产生空指针
         return Boolean.TRUE.equals(success);
     }
 
     @Override
     public void unlock() {
         stringRedisTemplate.delete(KEY_PREFIX + name);
     }
 }
 ~~~

使用分布式锁

~~~java
SimpleRedisLock simpleRedisLock = new SimpleRedisLock("order:" + userId, stringRedisTemplate);
boolean isLock = simpleRedisLock.tryLock(1200);
if(!isLock){
   //代表一个人在开挂抢！
   return Result.fail("不允许重复下单！");
}
try {
   VoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
   return proxy.createVoucherOrder(voucherId);
} catch (IllegalStateException e) {
   throw new RuntimeException(e);
} finally {
   simpleRedisLock.unlock();
}
~~~

### 误删问题

1. 持有锁的线程1在锁的内部出现了阻塞，导致他的锁TTL到期，自动释放
2. 此时线程2也来尝试获取锁，由于线程1已经释放了锁，所以线程2可以拿到
3. 但是现在线程1阻塞完了，继续往下执行，要开始释放锁了
4. 那么此时就会将属于线程2的锁释放，这就是误删别人锁的情况

解决方案就是在每个线程释放锁的时候，都**判断一下这个锁是不是自己的**，如果不属于自己，则不进行删除操作。

![image-20241019140127471](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241019140128.png)

改进思路

在获取锁的时候存入线程标识（用**UUID标识+线程id**，在一个JVM中，ThreadId一般不会重复，但是我们现在是集群模式，有多个JVM，多**个JVM之间可能会出现ThreadId重复的情况**），在释放锁的时候先获取锁的线程标识，判断是否与当前线程标识一致

- 如果一致则释放锁
- 如果不一致则不释放锁

~~~java
package com.hmdp.utils;

import cn.hutool.core.lang.UUID;
import org.springframework.data.redis.core.StringRedisTemplate;

import java.util.concurrent.TimeUnit;

public class SimpleRedisLock implements ILock{
    private static final String KEY_PREFIX = "lock:";
    private static final String ID_PREFIX = UUID.randomUUID().toString(true) + "-";
    //业务名
    private String name;
    //不是spring管理的类，不能自动给你注入
    private StringRedisTemplate stringRedisTemplate;

    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
        this.name = name;
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean tryLock(long timeoutSec) {
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        Boolean success = stringRedisTemplate
                .opsForValue().setIfAbsent(KEY_PREFIX + name, threadId, timeoutSec, TimeUnit.SECONDS);
        // 避免拆箱产生空指针
        return Boolean.TRUE.equals(success);
    }

    @Override
    public void unlock() {
        //获取线程标识
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        //获取锁里面存的标识
        String id = stringRedisTemplate.opsForValue().get(KEY_PREFIX + name);
        if(threadId.equals(id)){
            //释放锁
            stringRedisTemplate.delete(KEY_PREFIX + name);
        }
    }
}
~~~

![image-20241019141804179](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20241019141804179.png)

这样就完美了吗？还不是！！！

1. 假设线程1已经获取了锁，在**判断标识一致之后**，准备释放锁的时候，又出现了**阻塞（例如JVM垃圾回收机制）**
2. 于是锁的TTL到期了，自动释放了
3. 那么现在线程2趁虚而入，拿到了一把锁
4. 但是线程1的逻辑还没执行完，那么线程1就会执行删除锁的逻辑（**之前已经判断完了**）
5. 但是在阻塞前线程1已经判断了标识一致，所以现在线程1把线程2的锁给删了
6. 那么就相当于判断标识那行代码没有起到作用
7. 这就是删锁时的**原子性问题**
8. 因为线程1的拿锁，**判断标识，删锁，不是原子操作**，所以我们要防止刚刚的情况

![image-20241019150959186](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241019151000.png)

### Lua脚本解决原子性

Redis提供了Lua脚本功能，在一个脚本中编写多条Redis命令，确保多条命令执行时的原子性。

~~~bash
-- 这里的KEYS[1]就是传入锁的key
-- 这里的ARGV[1]就是线程标识
-- 比较锁中的线程标识与线程标识是否一致
if (redis.call('get', KEYS[1]) == ARGV[1]) then
    -- 一致则释放锁
    return redis.call('del', KEYS[1])
end
return 0
~~~

用Redis命令来调用脚本

```
EVAL script numkeys key [key ...] arg [arg ...]
```

~~~bash
EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 name Lucy
~~~

![image-20241019155015850](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241019155016.png)



~~~java
public class SimpleRedisLock implements ILock{
    private static final String KEY_PREFIX = "lock:";
    private static final String ID_PREFIX = UUID.randomUUID().toString(true) + "-";
    //业务名
    private String name;
    //不是spring管理的类，不能自动给你注入
    private StringRedisTemplate stringRedisTemplate;
    //lua脚本执行,静态变量加静态代码块，只会加载一次脚本
    private static final DefaultRedisScript<Long> UNLOCK_SCRIPT;
    static {
        UNLOCK_SCRIPT = new DefaultRedisScript<>();
        //加载资源
        UNLOCK_SCRIPT.setLocation(new ClassPathResource("unlock.lua"));
        UNLOCK_SCRIPT.setResultType(Long.class);
    }

    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
        this.name = name;
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean tryLock(long timeoutSec) {
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        Boolean success = stringRedisTemplate
                .opsForValue().setIfAbsent(KEY_PREFIX + name, threadId, timeoutSec, TimeUnit.SECONDS);
        // 避免拆箱产生空指针
        return Boolean.TRUE.equals(success);
    }

    @Override
    public void unlock() {
        //获取线程标识
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        //调用lua脚本
        stringRedisTemplate.execute(UNLOCK_SCRIPT,
                Collections.singletonList(KEY_PREFIX + name), threadId);

    }
~~~

![image-20241019161200947](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241019161202.png)

### 总结

**实现思路：**

- 利用set nx ex获取锁，并设置过期时间，保存线程标示
- 释放锁时先判断线程标示是否与自己一致，一致则删除锁。（Lua脚本保持原子性）

**Redis分布式锁特性：**

- 利用set nx满足互斥性
- 利用set ex保证故障时锁依然能释放，避免死锁，提高安全性
- 利用Redis集群保证高可用和高并发特性

## 可重入锁与不可重入锁

**可重入锁**：一个线程可以多次获得同一把锁，而不会导致死锁。每当线程获取锁时，它的重入计数增加；在释放锁时，计数减少，当计数为零时，锁才真正释放。 可以防止因同一线程未释放锁而引发的死锁问题。

Java 中的 `ReentrantLock`和`synchronized` 就是一个可重入锁。

~~~java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockExample {
    private final ReentrantLock lock = new ReentrantLock();

    public void methodA() {
        lock.lock();
        try {
            methodB(); // 可以重入
        } finally {
            lock.unlock();
        }
    }

    public void methodB() {
        lock.lock(); // 允许重入
        try {
            // do something
        } finally {
            lock.unlock();
        }
    }
}
~~~

**不可重入锁**：不可重入锁是指一个线程如果已经持有该锁，就无法再获得同一把锁。如果线程试图再次获取锁，就会导致死锁。（因为自己的业务没有完成，不会释放锁，又一直等待锁的释放）

## Redisson分布式锁

### 自己锁存在的问题

基于SETNX实现的分布式锁存在以下问题

1. 重入问题
   - 重入问题是指获取锁的线程，可以再次进入到相同的锁的代码块中，可重入锁的意义在于防止死锁，例如在HashTable这样的代码中，它的方法都是使用synchronized修饰的，加入它在一个方法内调用另一个方法，如果此时是不可重入的，那就死锁了。所以可重入锁的主要意义是防止死锁，我们的synchronized和Lock锁都是可重入的
2. 不可重试
   - 我们编写的分布式锁只能尝试一次，失败了就返回false，没有重试机制。但合理的情况应该是：当线程获取锁失败后，他应该能再次尝试获取锁。因此有些业务场景就不适合了，但刚才我们的一人一单是适合的。
3. 超时释放
   - 我们在加锁的时候增加了TTL，这样我们可以防止死锁，但是如果卡顿(阻塞)时间太长，超时也会导致锁的释放。虽然我们采用Lua脚本来防止删锁的时候，误删别人的锁，但问题是自己的锁没锁住，也有安全隐患。如果超时时间太长，自己宕机了，别人得等超时才能拿到锁，浪费时间。超时时间是个问题。
4. 主从一致性
   - 如果Redis提供了主从集群，那么当我们向集群写数据时，主机需要异步的将数据同步给从机，万一在同步之前，主机宕机了(主从同步存在延迟，虽然时间很短，但还是发生了)，那么又会出现线程不安全问题，因为其他线程在从机里没有看到锁。实际上发生概率极低，因为同步延迟很低。

### 什么是Redisson

​	Redisson是一个在Redis的基础上实现的Java驻内存数据网格(In-Memory Data Grid)。它不仅提供了一系列的分布式Java常用对象，还提供了许多分布式服务，其中就**包含了各种分布式锁的实现**：

- 可重入锁(Reentrant Lock)
- 公平锁(Fair Lock)
- 联锁(MultiLock)
- 红锁(RedLock)
- 读写锁(ReadWriteLock)
- 信号量(Semaphore)
- 可过期性信号量(PermitExpirableSemaphore)
- 闭锁(CountDownLatch)

### 使用方法

- 加Redisson依赖

```java
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.13.6</version>
</dependency>
```

- 配置Redisson客户端（连接地址，密码）

```java
@Configuration
public class RedissonConfig {
    @Bean
    public RedissonClient redissonClient(){
        Config config = new Config();
        config.useSingleServer().setAddress("redis://localhost:6379").setPassword("123");
        return Redisson.create(config);
    }
}
```

- 使用锁（获取锁对象、尝试获取锁、释放锁）

参数解释：

- `waitTime` 指的是线程在尝试获取锁时所愿意等待的最大时间。如果在这个时间内，锁仍然被其他线程持有，则该线程将放弃获取锁并返回 `false`。
- `leaseTime` 是指在成功获取到锁后，锁的有效时间（TTL）。在这个时间内，持有锁的线程必须完成其操作或者显式释放锁，否则锁会自动过期并被释放。

三种使用方法：

获取锁，如果什么都不传，则是获取锁失败后直接返回false，默认锁自动释放时间是30S；

如果传**三个参数**，第一个参数是waitTime等待时间，第二个是leaseTime（expire设置到redsi中），第三个是时间单位。

如果传入**两个参数**，第一个参数是waitTime等待时间，第二个参数是单位。leasetime为-1（锁自动释放时间为看门狗释放时间=30s）

注意：

只有当你使用 `tryLock(long waitTime, long leaseTime, TimeUnit unit)` 并将 `leaseTime` 设置为 `-1` 时，看门狗机制才会被激活，从而实现自动续约锁的功能。

![image-20241020154449930](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241020154451.png)

~~~java
 RLock lock = redissonClient.getLock("lock:order:" + userId);
        //第一个是等待时间，第二个是超市释放时间，第三个是时间单位
        //无参数的话，第一个是默认不等待，第二个是30s
        boolean isLock = lock.tryLock();
        if(!isLock){
            //代表一个人在开挂抢！
            return Result.fail("不允许重复下单！");
        }
        try {
            IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
            return proxy.createVoucherOrder(voucherId);
        } catch (IllegalStateException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
~~~

### 原理

#### 可重入

​	采用redis的**hash**结构来存储锁，其中**外层key**表示这把锁是否存在，**内层ke**y则记录当前这把锁被**哪个线程持有**，**value**存当前这个锁被**获取了几次**。

​	获取锁，value++，释放锁，value--，这二者都需要先判断这个锁是不是自己的。

​	获取和释放锁都需要重置更新次数，给业务留足够的时间。

​	为了保证**原子性**，所以流程图中的业务逻辑也是需要我们用**Lua**来实现的。

**获取锁**：

![image-20241020132603369](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241020132604.png)

**释放锁**：

如果判断出锁不是自己的，说明自己业务超时了，锁被释放了，不用管就行了，防止误删别人的锁。

![image-20241020132624139](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241020132625.png)

#### 可重试

获取锁成功，返回nil（java里面接受的是null）

获取锁失败，返回剩余到期时间

![image-20241020160318042](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241020160319.png)

利用了**信号量和订阅**，不是盲目尝试，是有别人释放锁了才尝试（如果此时有等待时间和锁未超时）

~~~java
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        long threadId = Thread.currentThread().getId();
        Long ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);
        //判断ttl是否为null
        if (ttl == null) {
            //获取锁成功
            return true;
        } else {
            //计算当前时间与获取锁时间的差值，让等待时间减去这个值
            time -= System.currentTimeMillis() - current;
            //如果消耗时间太长了，直接返回false，获取锁失败
            if (time <= 0L) {
                this.acquireFailed(waitTime, unit, threadId);
                return false;
            } else {
                //等待时间还有剩余，再次获取当前时间
                current = System.currentTimeMillis();
                //订阅别人释放锁的信号
                RFuture<RedissonLockEntry> subscribeFuture = this.subscribe(threadId);
                //在剩余时间内，没等到
                if (!subscribeFuture.await(time, TimeUnit.MILLISECONDS)) {
                    if (!subscribeFuture.cancel(false)) {
                        subscribeFuture.onComplete((res, e) -> {
                            if (e == null) {
                                //取消订阅
                                this.unsubscribe(subscribeFuture, threadId);
                            }

                        });
                    }
                    this.acquireFailed(waitTime, unit, threadId);
                    return false;
                } else {
                    try {
                        //如果剩余时间内等到了别人释放锁的信号，再次计算当前剩余最大等待时间
                        time -= System.currentTimeMillis() - current;
                        if (time <= 0L) {
                            //如果剩余时间为负数，则直接返回false
                            this.acquireFailed(waitTime, unit, threadId);
                            boolean var20 = false;
                            return var20;
                        } else {
                            boolean var16;
                            do {
                                //如果剩余时间等到了，dowhile循环重试获取锁！！！
                                long currentTime = System.currentTimeMillis();
                                ttl = this.tryAcquire(waitTime, leaseTime, unit, threadId);
                                //获取成功
                                if (ttl == null) {
                                    var16 = true;
                                    return var16;
                                }
                                time -= System.currentTimeMillis() - currentTime;
                                //超时了
                                if (time <= 0L) {
                                    this.acquireFailed(waitTime, unit, threadId);
                                    var16 = false;
                                    return var16;
                                }

                                currentTime = System.currentTimeMillis();
                                //信号量，等待时间是ttl和time的min
                                if (ttl >= 0L && ttl < time) {
                                    ((RedissonLockEntry)subscribeFuture.getNow()).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                                } else {
                                    ((RedissonLockEntry)subscribeFuture.getNow()).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
                                }

                                time -= System.currentTimeMillis() - currentTime;
                            } while(time > 0L);

                            this.acquireFailed(waitTime, unit, threadId);
                            var16 = false;
                            return var16;
                        }
                    } finally {
                        this.unsubscribe(subscribeFuture, threadId);
                    }
                }
            }
        }
    }
~~~

#### 超时续约

为什么需要超时释放？

如果某个操作耗时较长，而你设置的锁的过期时间（`leaseTime`）太短，操作在完成之前锁就会自动释放，其他线程或进程可能会获取到锁，从而导致并发访问共享资源，出现**数据不一致**或**竞态条件**。

如果你为了避免锁过期问题，将锁的持有时间设置得很长，万一持锁的节点意外宕机或网络中断，锁就可能会被“永久占用”，导致其他线程/进程无法获取锁，最终导致系统死锁或效率极低。

解决方案是通过**设置合理的锁过期时间**并配合**自动续约机制**：

- 锁的默认过期时间可以较短，例如 `30秒`。
- 如果线程正常执行任务并需要长时间持有锁，**看门狗机制**会自动续约，确保锁不会意外释放。
- 但如果持锁的线程异常退出或崩溃，锁将不会被续约，最终自动释放，避免死锁。

因此，我们应该定时刷新redis中存的有效期。

这时候**看门狗**就出现了，前提是**leaseTime为-1**，它会每隔 `10 秒`对锁进行续约，续约时间为 `30 秒`，直到锁被释放为止。



`EXPIRATION_RENEWAL_MAP`：这是一个 `Map` 类型的集合，用于存储所有活跃的锁和它们的续约信息。键是锁的名字，值是 `ExpirationEntry` 对象。

`ExpirationEntry`：保存与锁相关的线程 ID 和续约信息的对象，用来标识某个锁的线程是否已经设置了自动续约任务。

`threadId`：表示当前线程的 ID。



如果 `oldEntry` 不为空，说明 `EXPIRATION_RENEWAL_MAP` 中已经有了与该锁对应的续约任务。这通常发生在**线程重入**的情况下，即同一个线程再次获取同一个锁。

因为这个锁已经有自动续约任务在运行，因此无需重复启动新的定时任务，只需将当前线程的 `threadId` 添加到 `oldEntry` 中，表示该线程重入锁。

~~~java
private void scheduleExpirationRenewal(long threadId) {
    ExpirationEntry entry = new ExpirationEntry();  
    //不存在，才put，表明是第一次进入，不是重入
    ExpirationEntry oldEntry = (ExpirationEntry)EXPIRATION_RENEWAL_MAP.putIfAbsent(this.getEntryName(), entry);
    if (oldEntry != null) {
        //线程重入，不用再次调定时任务了，因为任务已经在重复运行了
        oldEntry.addThreadId(threadId);
    } else {
        //如果是第一次进入，则开启一个定时任务
        entry.addThreadId(threadId);
        this.renewExpiration();
    }
}
~~~

```java
private void renewExpiration() {
    ExpirationEntry ee = (ExpirationEntry)EXPIRATION_RENEWAL_MAP.get(this.getEntryName());
    if (ee != null) {
        //Timeout是一个定时任务
        Timeout task = this.commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
            public void run(Timeout timeout) throws Exception {
                ExpirationEntry ent = (ExpirationEntry)RedissonLock.EXPIRATION_RENEWAL_MAP.get(RedissonLock.this.getEntryName());
                if (ent != null) {
                    Long threadId = ent.getFirstThreadId();
                    if (threadId != null) {
                        //重置有效期
                        RFuture<Boolean> future = RedissonLock.this.renewExpirationAsync(threadId);
                        future.onComplete((res, e) -> {
                            if (e != null) {
                                RedissonLock.log.error("Can't update lock " + RedissonLock.this.getName() + " expiration", e);
                            } else {
                                if (res) {
                                    //然后调用自己，递归重置有效期
                                    RedissonLock.this.renewExpiration();
                                }

                            }
                        });
                    }
                }
            }
            //internalLockLeaseTime是之前WatchDog默认有效期30秒，那这里就是 30 / 3 = 10秒之后，才会执行
        }, this.internalLockLeaseTime / 3L, TimeUnit.MILLISECONDS);
        ee.setTimeout(task);
    }
}
```

~~~java
protected RFuture<Boolean> renewExpirationAsync(long threadId) {
    return this.evalWriteAsync(this.getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN, "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('pexpire', KEYS[1], ARGV[1]); return 1; end; return 0;", Collections.singletonList(this.getName()), this.internalLockLeaseTime, this.getLockName(threadId));
}
~~~

这个任务什么时候停止？自然是释放锁的时候停止

~~~java
void cancelExpirationRenewal(Long threadId) {
    //将之前的线程终止掉
    ExpirationEntry task = (ExpirationEntry)EXPIRATION_RENEWAL_MAP.get(this.getEntryName());
    if (task != null) {
        if (threadId != null) {
            task.removeThreadId(threadId);
        }

        if (threadId == null || task.hasNoThreads()) {
            //获取之前的定时任务
            Timeout timeout = task.getTimeout();
            if (timeout != null) {
                //取消
                timeout.cancel();
            }

            EXPIRATION_RENEWAL_MAP.remove(this.getEntryName());
        }

    }
}
~~~

#### 可重试和超时续约总结

这里的获取失败，是指别的线程占用了锁，不是自己的线程，因为可重入保证了自己线程占用也可以获取成功。

释放锁，也是指可重入意义上的释放锁，也就是value为0。

![image-20241020153202826](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241020153203.png)

Redisson分布式锁原理

- 可重入：利用hash结构记录线程id和重入次数
- 可重试：利用信号量和PubSub功能实现等待、唤醒，获取锁失败的重试机制，不浪费cpu资源
- 超时续约：利用watchD0g，每隔一段时间(releaseTime/3)，重置超时时间，也就是重新expire。

#### 主从一致性

​	在 Redis 主从架构下，锁通常是创建在主节点上的，因为**主节点负责写操作**。从节点则主要用于读操作。如果主节点宕机并且发生了主从切换（即从节点被提升为新的主节点），可能会出现以下问题：

- **锁丢失**：主节点上创建的锁无法及时同步到从节点。在主节点宕机之前的锁状态没有被复制，从节点不知道锁的存在，导致其他客户端可能错误地认为没有锁存在。
- **数据不一致**：由于 Redis 主从复制是异步的，因此在主节点发生故障之前，锁的状态可能尚未完全同步到从节点，造成分布式锁的**不可用**或**错误释放**

![image-20241020183914837](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241020183915.png)

解决方案：**联锁**

RedLock 通过在多个 Redis 实例（通常是 5 个）上**同时获取锁**，来确保即使某个 Redis 实例宕机，锁依然安全有效。

![image-20241020184023248](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241020184024.png)

![image-20241020184154743](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241020184155.png)

我们先使用虚拟机额外搭建两个Redis节点。

配置redis。

~~~java
@Configuration
public class RedissonConfig {
    @Bean
    public RedissonClient redissonClient(){
        Config config = new Config();
        config.useSingleServer().setAddress("redis://localhost:6379").setPassword("123");
        return Redisson.create(config);
    }

    @Bean
    public RedissonClient redissonClient2(){
        Config config = new Config();
        config.useSingleServer().setAddress("redis://192.168.140.132:6379").setPassword("123");
        return Redisson.create(config);
    }
}
~~~

使用MultiLock

~~~java
@Resource
    private RedissonClient redissonClient;
    @Resource
    private RedissonClient redissonClient2;
    private RLock lock;
    @BeforeEach
    void setUp() {
        RLock lock1 = redissonClient.getLock("lock");
        RLock lock2 = redissonClient2.getLock("lock");
        //创建联锁
        lock = redissonClient.getMultiLock(lock1, lock2);
    }

    @Test
    void method1() throws InterruptedException {
        boolean success = lock.tryLock(1,  TimeUnit.SECONDS);
        if (!success) {
            log.error("获取锁失败，1");
            return;
        }
        try {
            log.info("获取锁成功，1");
            Thread.sleep(200000);
            method2();
        } finally {
            log.info("释放锁，1");
            lock.unlock();
        }
    }

    void method2() {
        boolean success = lock.tryLock();
        if (!success) {
            log.error("获取锁失败，2");
            return;
        }
        try {
            log.info("获取锁成功，2");
        } finally {
            log.info("释放锁，2");
            lock.unlock();
        }
    }
~~~

可变参数，把你加入的全部Lock放入集合中

~~~java
public RedissonMultiLock(RLock... locks) {
    if (locks.length == 0) {
        throw new IllegalArgumentException("Lock objects are not defined");
    } else {
        //可变参数，把参数放入集合中
        this.locks.addAll(Arrays.asList(locks));
    }
}
~~~



~~~java
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
    long newLeaseTime = -1L;
    //如果传入了释放时间
    if (leaseTime != -1L) {
        //再判断一下是否有等待时间
        if (waitTime == -1L) {
            //如果没传等待时间，不重试，则只获得一次
            newLeaseTime = unit.toMillis(leaseTime);
        } else {
            //想要重试，耗时较久，万一释放时间小于等待时间，则会有问题，所以这里将等待时间乘以二
            newLeaseTime = unit.toMillis(waitTime) * 2L;
        }
    }
    //获取当前时间
    long time = System.currentTimeMillis();
    //剩余等待时间
    long remainTime = -1L;
    if (waitTime != -1L) {
        remainTime = unit.toMillis(waitTime);
    }
    //锁等待时间，与剩余等待时间一样    
    long lockWaitTime = this.calcLockWaitTime(remainTime);
    //锁失败的限制，源码返回是的0
    int failedLocksLimit = this.failedLocksLimit();
    //已经获取成功的锁
    List<RLock> acquiredLocks = new ArrayList(this.locks.size());
    //迭代器，用于遍历
    ListIterator<RLock> iterator = this.locks.listIterator();

    while(iterator.hasNext()) {
        RLock lock = (RLock)iterator.next();

        boolean lockAcquired;
        try {
            //没有等待时间和释放时间，调用空参的tryLock
            if (waitTime == -1L && leaseTime == -1L) {
                lockAcquired = lock.tryLock();
            } else {
                //否则调用带参的tryLock
                long awaitTime = Math.min(lockWaitTime, remainTime);
                //之前看过了
                lockAcquired = lock.tryLock(awaitTime, newLeaseTime, TimeUnit.MILLISECONDS);
            }
        } catch (RedisResponseTimeoutException var21) {
            this.unlockInner(Arrays.asList(lock));
            lockAcquired = false;
        } catch (Exception var22) {
            lockAcquired = false;
        }
        //判断获取锁是否成功
        if (lockAcquired) {
            //成功则将锁放入成功锁的集合
            acquiredLocks.add(lock);
        } else {
            //如果获取锁失败
            //判断当前锁的数量，减去成功获取锁的数量，如果为0，则所有锁都成功获取，跳出循环
            if (this.locks.size() - acquiredLocks.size() == this.failedLocksLimit()) {
                break;
            }
            //否则将拿到的锁都释放掉，全部推到重来
            if (failedLocksLimit == 0) {
                this.unlockInner(acquiredLocks);
                //如果等待时间为-1，则不想重试，直接返回false
                if (waitTime == -1L) {
                    return false;
                }

                failedLocksLimit = this.failedLocksLimit();
                //将已经拿到的锁都清空
                acquiredLocks.clear();
                //将迭代器往前迭代，相当于重置指针，放到第一个然后重试获取锁
                while(iterator.hasPrevious()) {
                    iterator.previous();
                }
            } else {
                --failedLocksLimit;
            }
        }
        //如果剩余时间不为-1，很充足
        if (remainTime != -1L) {
            //计算现在剩余时间
            remainTime -= System.currentTimeMillis() - time;
            time = System.currentTimeMillis();
            //如果剩余时间为负数，则获取锁超时了
            if (remainTime <= 0L) {
                //将之前已经获取到的锁释放掉，并返回false
                this.unlockInner(acquiredLocks);
                //联锁成功的条件是：每一把锁都必须成功获取，一把锁失败，则都失败
                return false;
            }
        }
    }
    //如果设置了锁的有效期
    if (leaseTime != -1L) {
        List<RFuture<Boolean>> futures = new ArrayList(acquiredLocks.size());
        //迭代器用于遍历已经获取成功的锁
        Iterator var24 = acquiredLocks.iterator();

        while(var24.hasNext()) {
            RLock rLock = (RLock)var24.next();
            //设置每一把锁的有效期，为什么需要设置？因为没使用看门狗
            RFuture<Boolean> future = ((RedissonLock)rLock).expireAsync(unit.toMillis(leaseTime), TimeUnit.MILLISECONDS);
            futures.add(future);
        }

        var24 = futures.iterator();

        while(var24.hasNext()) {
            RFuture<Boolean> rFuture = (RFuture)var24.next();
            rFuture.syncUninterruptibly();
        }
    }
    //但如果没设置有效期，则会触发WatchDog机制，自动帮我们设置有效期，所以大多数情况下，我们不需要自己设置有效期
    return true;
}
~~~

#### 再次总结

1. **不可重入Redis分布式锁**:

   原理:利用setnx的互斥性;利用ex避免死锁;释放锁时判断线程标示

   缺陷:不可重入、无法重试、锁超时失效

2. **可重入的Redis分布式锁**:

   原理:利用hash结构，记录线程标示和重入次数;利用watchDog延续锁时间;利用信号量控制锁重试等待

   缺陷:redis宕机引起锁失效问题

3. **Redisson的multiLock**:

   原理:多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取锁成功

   缺陷:运维成本高、实现复杂

其中multiLock可以看做多个Redis分布式锁。

## Redis优化秒杀

原逻辑：速度慢

- 一个线程串行执行
- 很多操作都是要去操作数据库的
- 使用了悲观锁：分布式锁

![image-20241021113033762](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241021113034.png)



核心：**读写分离**

​	读取的操作比较快，写入的操作比较慢，我们就可以分开处理。

​	我们将耗时较短的**查询**逻辑判断放到Redis中，例如：库存是否充足，是否一人一单这样的操作，只要满足这两条操作，那我们是一定可以下单成功的，不用等数据真的写进数据库，因为这样实际上用户已经完成了下单，他获得了订单号，只要去支付就行了。

​	然后后台再开一个线程，后台线程再去读取阻塞队列里的消息，执行操作慢的数据库写操作。

![image-20241021113320476](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241021113321.png)

​	库存可以用**String**存，可以用**Set**存这个优惠卷被哪些用户下了单。

​	因为这是多个操作，需要保证操作的**原子性**，可以使用Lua脚本。

![image-20241021113546012](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241021113547.png)

- 新增秒杀优惠券的同时，将优惠券信息保存到Redis中
- 基于Lua脚本，判断秒杀库存、一人一单，决定用户是否抢购成功
- 如果抢购成功，将优惠券id和用户id封装后存入阻塞队列
- 开启线程任务，不断从阻塞队列中获取信息，实现异步下单功能

lua脚本

~~~lua
-- 订单id
local voucherId = ARGV[1]
-- 用户id
local userId = ARGV[2]
-- 优惠券key
local stockKey = 'seckill:stock:' .. voucherId
-- 订单key
local orderKey = 'seckill:order:' .. voucherId
-- 判断库存是否充足
if (tonumber(redis.call('get', stockKey)) <= 0) then
    return 1
end
-- 判断用户是否下单
if (redis.call('sismember', orderKey, userId) == 1) then
    return 2
end
-- 扣减库存
redis.call('incrby', stockKey, -1)
-- 将userId存入当前优惠券的set集合
redis.call('sadd', orderKey, userId)
return 0
~~~

~~~java
@Service
public class VoucherOrderServiceImpl extends ServiceImpl<VoucherOrderMapper, VoucherOrder> implements IVoucherOrderService {

    @Resource
    private ISeckillVoucherService seckillVoucherService;
    @Resource
    private RedisIdWorker redisIdWorker;
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Resource
    private RedissonClient redissonClient;

    private BlockingQueue<VoucherOrder> orderTasks = new ArrayBlockingQueue<>(1024 * 1024);
    private static final ExecutorService SECKILL_ORDER_EXECUTOR = Executors.newSingleThreadExecutor();

    //秒杀业务需要在类初始化之后，就立即执行
    @PostConstruct
    private void init(){
        //开启了独立的子线程
        SECKILL_ORDER_EXECUTOR.submit(new VoucherOrderHandler());
    }
    private IVoucherOrderService proxy;
    private class VoucherOrderHandler implements Runnable{
        @Override
        public void run() {
            while(true){
                try {
                    //如果队列为空就会等待
                    VoucherOrder voucherOrder = orderTasks.take();
                    proxy.createVoucherOrder(voucherOrder);
                } catch (InterruptedException e) {
                    throw new RuntimeException("处理订单异常");
                }
            }
        }
    }

    @Transactional
    //单线程不会发生并发问题，即使是多线程，只执行update语句也不会有问题
    public void createVoucherOrder(VoucherOrder voucherOrder) {
        // 扣减库存,乐观锁
       seckillVoucherService.update()
                .setSql("stock = stock - 1")
                .eq("voucher_id", voucherOrder.getVoucherId())
                .gt("stock",0)
                .update();
        // 创建订单
        save(voucherOrder);
    }

    //lua脚本执行,静态变量加静态代码块，只会加载一次脚本
    private static final DefaultRedisScript<Long> SECKILL_SCRIPT;
    static {
        SECKILL_SCRIPT = new DefaultRedisScript<>();
        //加载资源
        SECKILL_SCRIPT.setLocation(new ClassPathResource("seckill.lua"));
        SECKILL_SCRIPT.setResultType(Long.class);
    }

    @Override
    public Result seckillVoucher(Long voucherId) {
        Long userId = UserHolder.getUser().getId();
        //1执行脚本,第一个参数是脚本，第二个参数是KEY的list结合，第三个参数是可变参数（其他变量）
        Long isSuccess = stringRedisTemplate.execute(
                SECKILL_SCRIPT,
                Collections.emptyList(),
                voucherId.toString(),
                userId.toString()
        );
        //2如果返回的不是0，表示下单失败
        if(isSuccess == 1L){
            return Result.fail("库存不足！");
        } else if(isSuccess == 2L){
            return Result.fail("不允许重复下单！");
        }
        long orderId = redisIdWorker.nextId("order");
        //3 订单加入到阻塞队列
        VoucherOrder voucherOrder = new VoucherOrder(orderId, userId, voucherId);
        //子线程看不到代理对象，因此把代理对象弄成成员变量。
        proxy = (IVoucherOrderService) AopContext.currentProxy();
        orderTasks.add(voucherOrder);
        return Result.ok(orderId);
    }
}
~~~

存在的问题？

1. 内存限制问题：我们现在使用的是JDK里的阻塞队列，它使用的是JVM的内存，如果在高并发的条件下，无数的订单都会放在阻塞队列里，可能就会造成内存溢出，所以我们在创建阻塞队列时，设置了一个长度，但是如果真的存满了，再有新的订单来往里塞，那就塞不进去了，存在内存限制问题
2. 数据安全问题：服务器宕机了，JVM没有持久化机制，阻塞队列数据消失，用户明明下单了，但是数据库里没看到

## 消息队列优化秒杀

### 什么是消息队列

什么是消息队列？字面意思就是存放消息的队列，最简单的消息队列模型包括3个角色

1. 消息队列：存储和管理消息，也被称为消息代理（Message Broker）
2. 生产者：发送消息到消息队列
3. 消费者：从消息队列获取消息并处理消息

消息队列还应该：

1. 不受JVM内存限制，可以存很多东西。
2. 有持久化机制，服务宕机还是重启，数据不会消失。
3. 有消息确认机制，没有确认的消息会在队列一直存在，直到消费者确认。
4. 有序性，先来的消息要先处理。

​	使用消息队列的好处在于**解耦**：举个例子，快递员(生产者)把快递放到驿站/快递柜里去(Message Queue)去，我们(消费者)从快递柜/驿站去拿快递，这就是一个异步，如果耦合，那么快递员必须亲自上楼把快递递到你手里，服务当然好，但是万一我不在家，快递员就得一直等我，浪费了快递员的时间。所以解耦还是非常有必要的。

1. 那么在这种场景下我们的秒杀就变成了：在我们下单之后，利用Redis去进行校验下单的结果，然后在通过队列把消息发送出去，然后在启动一个线程去拿到这个消息，完成解耦，同时也加快我们的响应速度
2. 这里我们可以直接使用一些现成的(MQ)消息队列，如kafka，rabbitmq等，但是如果没有安装MQ，我们也可以使用Redis提供的MQ方案(学完Redis我就去学微服务)

![image-20241021165051040](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241021165052.png)

Redis提供了三种不同的方式来实现消息队列:

- list结构:基于List结构模拟消息队列
- Pubsub:基本的点对点消息模型
- Stream:比较完善的消息队列模型

![image-20241022094610406](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241022094611.png)

### List模拟消息队列

![image-20241021170456356](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241021170457.png)



- 优点
  1. 利用Redis存储，不受限于JVM内存上限
  2. 基于Redis的持久化机制，数据安全性有保障
  3. 可以满足消息有序性
- 缺点
  1. 无法避免消息丢失，你取走了消息，List数据没了，但是你挂了，消息永远也没了
  2. 只支持单消费者(一个消费者把消息拿走了，其他消费者就看不到这条消息了)

### PubSub消息队列

![image-20241021175202782](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241021175203.png)

- 优点：
  1. 采用发布订阅模型，支持多生产，多消费
- 缺点：
  1. 不支持**数据持久化**，list支持是因为list本身就是一个数据结构。
  2. 无法避免消息丢失（如果向频道发送了消息，却没有人订阅该频道，那发送的这条消息就丢失了）
  3. 消息堆积有上限（消费者的缓存（内存）是有限的）

### Stream的单消费者模式

Stream是新**数据类型**

发送消息：

![image-20241021181715815](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241021181716.png)

默认是，不阻塞读取，读不到就返回nal

![image-20241021182258317](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241021182259.png)

![image-20241021182601567](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241021182602.png)

- STREAM类型消息队列的XREAD命令特点
  1. 消息可回溯，读取后不消失，永久的保存在队列中
  2. 一个消息可以被多个消费者读取
  3. 可以阻塞读取
  4. 有漏读消息的风险

### Stream的消费者组

1. 消息分流
   - 队列中的消息会分留给组内的不同消费者，而不是重复消费者，**从而加快消息处理的速度**。如果想重复消费，可以创建多个消费者组。
2. 消息标识
   - 消费者组会维护一个标识，记录组内**最后一个被处理的消息**，哪怕消费者宕机重启，还会从标识之后读取消息，确保每一个消息都会被消费。**注意组内只会维护一个标记。**
3. 消息确认
   - 消费者获取消息后，消息处于**pending**状态，并存入一个pending-list，当处理完成后，需要通过XACK来确认消息，标记消息为已处理，才会从pending-list中移除。这样即使宕机，也可以在pending-list中读取消息。每个消费者有自己的pending-list。消费者组也会维护一个共同的pending-list。

**创建消费者组**

可以看出流（队列）和消费者组是多对多的关系

~~~bash
XGROUP CREATE key groupName ID [MKSTREAM]
XGROUP create s1 g1 0 MKSTREAM
~~~

- key：流的名称
- groupName：消费者组名称
- ID：起始ID标识，代表队列中的最后一个消息，0代表队列中的第一个消息。如果队列之前有消息，不想要可以选择使用$。
- MKSTREAM：队列不存在时自动创建队列

**删除消费者组**

```txt
XGROUP DESTORY key groupName
XGROUP DESTROY s1 g1
```

- key：流的名称
- groupName：消费者组名称

**向队列添加消息**

~~~txt
XADD key [NOMKSTREAM] [MAXLEN|MINID [=|~] threshold [LIMIT count]] *|ID field value [field value ...] 
XADD s1 * k1 v1
~~~

- key：队列名称
- NOMKSTREAM:队列没有的话自动创建队列，默认没有队列自动创建。
- [MAXLEN|MINID [=|~] threshold [LIMIT count]] ：设置消息队列的最大消息数量，不设置则无上限
- `*`|ID：`*`代表由Redis自动生成。格式是”时间戳-递增数字”，例如”114514114514-0”
- field value [field value …]: 发送到队列中的消息，称为Entry。格式就是**多个key-value键值对**

![image-20241022092309995](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241022092311.png)

**从消费者组读取消息**

~~~txt
XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [keys ...] ID [ID ...]
XREADGROUP group g1 c1 count 1 block 200 STREAMS s1 >
~~~

- group：消费者组名称
- consumer：消费者名，如果消费者不存在，会自动创建一个消费者
- count：本次查询的最大消息数量
- BLOCK milliseconds：当前没有消息时的最大等待时间
- NOACK：无需手动ACK，获取到消息后自动确认（一般不用，我们都是手动确认）
- STREAMS key：指定队列名称
- ID获取消息的起始ID
  - `>`：从下一个未消费的消息开始，正常情况下用这个。
  - 其他：根据指定id从pending-list中获取已消费但未确认的消息，例如0，是从pending-list中的第一个消息开始，异常情况下从这里开始。

**专门读取未确认的消息**

~~~txt
XPENDING group [[IDLE min-idle-time] start end count [consumer]]
XPENDING g1 - + 10
~~~

- group：消费者组
- start end：id范围，如果是 - +，表示所有范围我都要
- count：取几个消息
- consumer： 消费者，不指定就是整个消费者组的未确认消息

**确认消息**

~~~java
XACK key group ID [ID ...]    
XACK s1 g1 1729558781707-0
~~~

- key：队列名称
- group：消费者组
- ID：消息的id

**伪代码**

![image-20241022094020159](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241022094021.png)

**优点**

- 消息可回溯
- 可以多消费者争抢消息，加快消费速度
- 可以阻塞读取
- 没有消息漏读的风险
- 有消息确认机制，保证消息至少被消费一次

**缺点**

相比于专业的消息中间件

- 依赖于Redis，不能保证万无一失。
- 没有生产者确认机制。

~~~lua
-- 订单id
local voucherId = ARGV[1]
-- 用户id
local userId = ARGV[2]
-- 新增orderId，但是变量名用id就好，因为VoucherOrder实体类中的orderId就是用id表示的
local id = ARGV[3]
-- 优惠券key
local stockKey = 'seckill:stock:' .. voucherId
-- 订单key
local orderKey = 'seckill:order:' .. voucherId
-- 判断库存是否充足
if (tonumber(redis.call('get', stockKey)) <= 0) then
    return 1
end
-- 判断用户是否下单
if (redis.call('sismember', orderKey, userId) == 1) then
    return 2
end
-- 扣减库存
redis.call('incrby', stockKey, -1)
-- 将userId存入当前优惠券的set集合
redis.call('sadd', orderKey, userId)
-- 将下单数据保存到消息队列中
redis.call("xadd", 'stream.orders', '*', 'voucherId', voucherId, 'userId', userId, 'id', id)
return 0
~~~



~~~java
 private class VoucherOrderHandler implements Runnable{
        @Override
        public void run() {
            while(true){
                try {
                    //1. 获取消息队列中的订单信息 XREADGROUP GROUP g1 c1 COUNT 1 BLOCK 2000 STREAMS stream.orders >
                    List<MapRecord<String, Object, Object>> list = stringRedisTemplate.opsForStream().read(
                            Consumer.from("g1", "c1"),
                            StreamReadOptions.empty().count(1).block(Duration.ofSeconds(2)),
                            StreamOffset.create("stream.orders", ReadOffset.lastConsumed())
                    );
                    //2 获取失败，继续下次循环
                    if(list == null || list.isEmpty()){
                        continue;
                    }
                    handleVoucherOrder(list);
                } catch (Exception e) {
                    log.error("处理消息异常！");
                    handlePendingList();
                }
            }
        }

        private void handlePendingList() {
            while(true){
                try {
                    //1. 获取pending-list中的订单信息 XPENDING stream.orders g1 - + 10
                    List<MapRecord<String, Object, Object>> list = stringRedisTemplate.opsForStream().read(
                            Consumer.from("g1", "c1"),
                            StreamReadOptions.empty().count(1),
                            StreamOffset.create("stream.orders", ReadOffset.from("0"))
                    );
                    //2 没有消息，结束
                    if(list == null || list.isEmpty()){
                        break;
                    }
                    //3 有消息，处理消息
                    handleVoucherOrder(list);
                } catch (Exception e) {
                    log.error("处理pending-list消息异常！", e);
                    //休眠一会
                    try {
                        Thread.sleep(50);
                    } catch (InterruptedException ex) {
                        throw new RuntimeException(ex);
                    }
                }
            }
        }

        private void handleVoucherOrder(List<MapRecord<String, Object, Object>> list) {
            //因为我们只取一个消息
            MapRecord<String, Object, Object> record = list.get(0);
            //消息的id
            RecordId recordId = record.getId();
            //我们放进去的键值对
            Map<Object, Object> value = record.getValue();
            //处理消息：创建订单
            VoucherOrder voucherOrder = BeanUtil.fillBeanWithMap(value, new VoucherOrder(), true);
            proxy.createVoucherOrder(voucherOrder);
            //ack确认!!
            stringRedisTemplate.opsForStream().acknowledge("stream.orders", "g1", recordId);
        }
    }
~~~

## 达人探店

### 发布探店笔记

~~~java
 @PostMapping
    public Result saveBlog(@RequestBody Blog blog) {
        // 获取登录用户
        UserDTO user = UserHolder.getUser();
        blog.setUserId(user.getId());
        // 保存探店博文
        blogService.save(blog);
        // 返回id
        return Result.ok(blog.getId());
    }
~~~

### 查看探店笔记

~~~java
public Result queryBlogById(Long id) {
        Blog blog = this.getById(id);
        if(blog == null){
            return Result.fail("笔记不存在！");
        }
        // 查询blog有没有被当前用户点，赋值userIcon和name
        queryBlogUser(blog);
        // 查询blog有没有被当前用户点赞，赋值isLiked
        isBlogLiked(blog);
        return Result.ok(blog);
    }

    private void queryBlogUser(Blog blog) {
        Long userId = blog.getUserId();
        User user = userService.getById(userId);
        blog.setName(user.getNickName());
        blog.setIcon(user.getIcon());
    }
~~~

### 点赞功能

- 需求
  1. 同一个用户只能对同一篇笔记点赞一次，再次点击则取消点赞
  2. 如果当前用户已经点赞，则点赞按钮高亮显示（前端已实现，判断字段Blog类的isLike属性）
- 实现步骤
  1. 修改点赞功能，利用**Redis中的set**集合来判断是否点赞过，未点赞则点赞数`+1`，已点赞则点赞数`-1`
  2. 修改根据id查询的业务，判断当前登录用户是否点赞过，赋值给isLike字段
  3. 修改分页查询Blog业务，判断当前登录用户是否点赞过，赋值给isLike字段

为什么不用数据库，加一张表，存着是店铺id和用户id？

没必要，对数据库压力大，redis就可以。

**点赞功能：**

~~~java
 public Result likeBlog(Long id) {
        Long userId = UserHolder.getUser().getId();
        String key = RedisConstants.BLOG_LIKED_KEY + id;
        // 1.判断当前登录用户有没有点赞
        Boolean isLike = stringRedisTemplate.opsForSet().
                isMember(key, userId.toString());
        // 2已经点赞了，取消赞
        if(BooleanUtil.isTrue(isLike)){
            // 2.1修改数据库
            boolean isSuccess = update().setSql("liked = liked - 1").eq("id", id).update();
            // 2.2redis的set集合中删除
            if(isSuccess){
                stringRedisTemplate.opsForSet().remove(key, userId.toString());
            }
        } else{
            // 3 没有点赞
            // 3.1修改数据库
            boolean isSuccess = update().setSql("liked = liked + 1").eq("id", id).update();
            // 3.2增加到redis
            if(isSuccess){
                stringRedisTemplate.opsForSet().add(key, userId.toString());
            }
        }
        return Result.ok();
    }
~~~

**查询blog功能**

~~~java
 @Override
    public Result queryBlogById(Long id) {
        Blog blog = this.getById(id);
        if(blog == null){
            return Result.fail("笔记不存在！");
        }
        // 查询blog有没有被当前用户点，赋值userIcon和name
        queryBlogUser(blog);
        // 查询blog有没有被当前用户点赞，赋值isLiked
        isBlogLiked(blog);
        return Result.ok(blog);
    }


private void isBlogLiked(Blog blog){
        Long userId = UserHolder.getUser().getId();
        String key = RedisConstants.BLOG_LIKED_KEY + blog.getId();
        // 1.判断当前登录用户有没有点赞
        Boolean isLike = stringRedisTemplate.opsForSet().
                isMember(key, userId.toString());
        // 2.根据点赞信息赋值
        blog.setIsLike(BooleanUtil.isTrue(isLike));
    }
~~~

### 点赞排行榜

按照时间排序:把之前的set改为sortset。

~~~java
private void isBlogLiked(Blog blog){
        UserDTO userDTO = UserHolder.getUser();
        if(userDTO == null){
            //用户未登录，无需查询
            return;
        }
        Long userId = userDTO.getId();
        String key = RedisConstants.BLOG_LIKED_KEY + blog.getId();
        // 1.判断当前登录用户有没有点赞
        Double score = stringRedisTemplate.opsForZSet().score(key, userId.toString());
        // 2.根据点赞信息赋值
        blog.setIsLike(score != null);
    }


 public Result likeBlog(Long id) {
        Long userId = UserHolder.getUser().getId();
        String key = RedisConstants.BLOG_LIKED_KEY + id;
        // 1.判断当前登录用户有没有点赞
        Double score = stringRedisTemplate.opsForZSet().score(key, userId.toString());
        // 2已经点赞了，取消赞
        if(score != null){
            // 2.1修改数据库
            boolean isSuccess = update().setSql("liked = liked - 1").eq("id", id).update();
            // 2.2redis的set集合中删除
            if(isSuccess){
                stringRedisTemplate.opsForZSet().remove(key, userId.toString());
            }
        } else{
            // 3 没有点赞
            // 3.1修改数据库
            boolean isSuccess = update().setSql("liked = liked + 1").eq("id", id).update();
            // 3.2增加到redis
            if(isSuccess){
                stringRedisTemplate.opsForZSet().add(key, userId.toString(), System.currentTimeMillis());
            }
        }
        return Result.ok();
    }
~~~

注意，mysql查询使用in语句 ，比如`in(5,4,3,2,1)`，mysql是按照`1 2 3 4 5`去查询。如果想按照给的顺序查询，需要:

`order by field(id, 5, 4, 3, 2, 1)`

~~~java
public Result queryBlogLikes(Long id) {
        String key = RedisConstants.BLOG_LIKED_KEY + id;
        //前五名
        Set<String> top5 = stringRedisTemplate.opsForZSet().range(key, 0, 4);
        if(top5 == null || top5.isEmpty()){
            return Result.ok(Collections.emptyList());
        }
        List<Long> ids = top5.stream().map(Long::valueOf).collect(Collectors.toList());
        //这里注意， where id in (5, 1) 会乱序，会按照你给定in集合的字典序排序，而不是你指定的顺序
        //可以使用order by field(id, 5, 1)排序
//        List<User> users = userService.listByIds(ids);
        String idStr = StrUtil.join(",", ids);
        List<User> users = userService.query()
                .in("id", ids)
                .last("order by field ( id, " + idStr + ")")
                .list();
        List<UserDTO> userDTOS = BeanUtil.copyToList(users, UserDTO.class);
        return Result.ok(userDTOS);
    }
~~~

## 好友关注

### 关注

直接插入数据库就行，删除就是删除数据库

~~~java
 public Result follow(Long followUserId, Boolean isFollow) {
        Long userId = UserHolder.getUser().getId();
        if(isFollow){
            //新增关注
            Follow follow = new Follow(userId, followUserId);
            save(follow);
        } else{
            //取消关注
            QueryWrapper<Follow> queryWrapper = new QueryWrapper<Follow>()
                    .eq("user_id", userId)
                    .eq("follow_user_id", followUserId);
            remove(queryWrapper);
        }
        return Result.ok();
    }
~~~

### 共同关注

用redis中两个set求交集就可以。

~~~java
public Result follow(Long followUserId, Boolean isFollow) {
        Long userId = UserHolder.getUser().getId();
        String key = RedisConstants.FOLLOWS_KEY + userId;
        if(isFollow){
            //新增关注
            Follow follow = new Follow(userId, followUserId);
            boolean isSuccess = save(follow);
            if(isSuccess){
                stringRedisTemplate.opsForSet().add(key, followUserId.toString());
            }
        } else{
            //取消关注
            QueryWrapper<Follow> queryWrapper = new QueryWrapper<Follow>()
                    .eq("user_id", userId)
                    .eq("follow_user_id", followUserId);
            boolean isSuccess = remove(queryWrapper);
            if(isSuccess){
                stringRedisTemplate.opsForSet().remove(key, followUserId.toString());
            }
        }
        return Result.ok();
    }


public Result followCommons(Long id) {
        //1.获取当前用户
        Long userId = UserHolder.getUser().getId();
        //2.求交集
        String key1 = RedisConstants.FOLLOWS_KEY + userId;
        String key2 = RedisConstants.FOLLOWS_KEY + id;
        Set<String> intersect = stringRedisTemplate.opsForSet().intersect(key1, key2);
        //3.交集为空，返回空对象
        if(intersect == null){
            return Result.ok(Collections.emptyList());
        }
        //4.解析用户id
        List<Long> userIds = intersect.stream().map(Long::valueOf).collect(Collectors.toList());
        //5.查询用户
        List<User> users = userService.listByIds(userIds);
        //6.封装DTO
        List<UserDTO> userDTOS = BeanUtil.copyToList(users, UserDTO.class);
        return Result.ok(userDTOS);
    }
~~~

## Feed流

### 两种模式

1. Timeline：不做内容筛选，简单的按照内容发布时间排序，常用于好友或关注(B站关注的up，朋友圈等)
   - 优点：信息全面，不会有缺失，并且实现也相对简单
   - 缺点：信息噪音较多，用户不一定感兴趣，内容获取效率低
2. 智能排序：利用智能算法屏蔽掉违规的、用户不感兴趣的内容，推送用户感兴趣的信息来吸引用户
   - 优点：投喂用户感兴趣的信息，用户粘度很高，容易沉迷
   - 缺点：如果算法不精准，可能会起到反作用（给你推的你都不爱看）

### TimeLine的三种实现

**拉模式**

也叫读扩散，

- 该模式的核心含义是：当张三和李四、王五发了消息之后，都会保存到自己的发件箱中，如果赵六要读取消息，那么他会读取他自己的收件箱，此时系统会从他关注的人群中，将他关注人的信息全都进行拉取，然后进行排序
- 优点：比较**节约空间**，因为赵六在读取信息时，并**没有重复读取**，并且读取完之后，可以将他的收件箱清除
- 缺点：有延迟，当用户读取数据时，才会去关注的人的时发件箱中拉取信息，假设**该用户关注了海量用户**，那么此时就会拉取很多信息，**对服务器压力巨大**

![image-20241025105408296](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241025105411.png)

**推模式**

也叫写扩散

- 推模式是没有写邮箱的，当张三写了一个内容，此时会主动把张三写的内容发送到它粉丝的收件箱中，假设此时李四再来读取，就不用再去临时拉取了
- 优点：**时效快，不用临时拉取**
- 缺点：内存压力大，假设一个大V发了一个动态，很多人关注他，那么就会**写很多份数据到粉丝那边去**

![image-20241025105553803](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241025105554.png)



**推拉结合**

推拉模式是一个折中的方案。

- 站在发件人这一边，如果是普通人，那么我们采用写扩散的方式，直接把数据写入到他的粉丝收件箱中，因为普通人的粉丝数量较少，所以这样不会产生太大压力。
- 但如果是大V，那么他是直接将数据写入一份到发件箱中去，在直接写一份到活跃粉丝的收件箱中。

- 站在收件人这边来看，如果是活跃粉丝，那么大V和普通人发的都会写到自己的收件箱里。
- 但如果是普通粉丝，由于上线不是很频繁，所以等他们上线的时候，再从发件箱中去拉取信息。

![image-20241025105830071](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241025105831.png)

![image-20241025105901702](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241025105902.png)

我们的点评网，没有过千万粉丝，用户量少，可以使用推模式。

### 推送实现

~~~java
public Result saveBlog(Blog blog) {
        //1.获取登录用户
        UserDTO userDTO = UserHolder.getUser();
        blog.setUserId(userDTO.getId());
        //2.保存探店博客
        boolean isSuccess = save(blog);
        if(!isSuccess){
            return Result.fail("新增笔记失败");
        }
        //3查询当前登录用户的粉丝
        List<Follow> follows = followService.query().eq("follow_user_id", userDTO.getId()).list();
        for (Follow follow : follows) {
            //4.推送
            String key = RedisConstants.FEED_KEY + follow.getUserId();
            stringRedisTemplate.opsForZSet().add(key, blog.getId().toString(), System.currentTimeMillis());
        }
        return Result.ok(blog.getId());
    }
~~~

### 滚动分页查询理论

由于我们的feed流是动态添加的，不能使用传统的按角标方式进行查询，否则会查重复。

![image-20241026152930890](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241026152932.png)

我们需要记录每次操作的最后一条，然后从这个位置去开始读数据。

- 我们需要记录每次操作的最后一条，然后从这个位置去开始读数据
- 举个例子：我们从t1时刻开始，拿到第一页数据，拿到了`10~6`，然后记录下当前最后一次读取的记录，就是6，t2时刻发布了新纪录，此时这个11在最上面，但不会影响我们之前拿到的6，此时t3时刻来读取第二页，第二页读数据的时候，从`6-1=5`开始读，这样就拿到了`5~1`的记录。我们在这个地方可以使用**SortedSet**来做，使用**时间戳**来充当表中的`1~10`

![image-20241026153208367](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241026153209.png)

因此不能使用SortedSet 的 Range语句，这也是用下表查询，使用的是

~~~txt
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
~~~

- ZREVRANGEBYSCORE ：降序按照得分查，**注意我们需要的是按照时间戳降序查，越新的blog时间戳越大。**
- key： sortedset的键
- max：最大得分
- min：最小得分
- [WITHSCORES]：是否显示分数
- [LIMIT offset count]：offset：从最大得分的第几位开始查，count表示查询几个



**如果是第一次查询，max给当前时间戳，offset填0就行。**

**如果不是第一次查询：max给上一次查询的最小时间戳，offset给上一次查询结果中，与最小时间戳一样元素的个数。**

min不需要管，count是我们自己设计的。

![image-20241026153921232](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241026153922.png)

### 滚动分页查询实现

第一次进入，后台没有返回给你数据，自然就是没有offset。

后面你进行滚动，**滚到底触发一次新的查询（后端返回给你此页的数据，业内滚动不触发查询）**，查询中带上了上一次返回给你的offset和lastId。

![image-20241026154126382](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241026154127.png)

~~~java
 @GetMapping("/of/follow")
    public Result queryBlogOfFollow(@RequestParam("lastId") Long max, @RequestParam(value = "offset", defaultValue = "0") Integer offset){
        return blogService.queryBlogOfFollow(max, offset);
    }

@Override
    public Result queryBlogOfFollow(Long max, Integer offset) {
        //1.获取当前用户
        UserDTO userDTO = UserHolder.getUser();
        //2.去收件箱里查询
        String key = RedisConstants.FEED_KEY + userDTO.getId();
        Set<ZSetOperations.TypedTuple<String>> typedTuples = stringRedisTemplate.opsForZSet()
                .reverseRangeByScoreWithScores(key, 0, max, offset, 2);
        //3.判断是否有blog
        if(typedTuples == null || typedTuples.isEmpty()){
            return Result.ok();
        }
        //4.解析
        ArrayList<Long> ids = new ArrayList<>(typedTuples.size());
        long minScore = 0;
        int cnt = 1;
        for (ZSetOperations.TypedTuple<String> typedTuple : typedTuples) {
            //存进去的blogId
            ids.add(Long.valueOf(typedTuple.getValue()));
            //时间戳
            long score = typedTuple.getScore().longValue();
            if(score == minScore){
                cnt ++;
            } else{
                minScore = score;
                cnt = 1;
            }
        }
        //5.根据ids去查询blog，注意必须要有序
        String idStr = StrUtil.join(",", ids);
        List<Blog> blogs = query()
                .in("id", ids)
                .last("order by field ( id, " + idStr + ")")
                .list();
        //6.关联信息
        for (Blog blog : blogs) {
            // 赋值userIcon和name
            queryBlogUser(blog);
            // 查询blog有没有被当前用户点赞，赋值isLiked
            isBlogLiked(blog);
        }
        //7.封装scrollResult
        ScrollResult scrollResult = new ScrollResult(blogs, minScore, cnt);
        return Result.ok(scrollResult);
    }
~~~

## 附近商户

### GEO结构

GEO就是Geolocation的简写形式，代表地理坐标。

Redis在3.2版本中加入了对GEO的支持，允许存储地理坐标信息，帮助我们根据经纬度来检索数据。

**内部使用Zset实现。**

### **常用命令**

**GEOADD**：添加一个地理空间信息，包含：经度（longitude）、纬度（latitude）、值（member）

- 返回值：添加到sorted set元素的数目，但不包括已更新score的元素
- 复杂度：每⼀个元素添加是O(log(N)) ，N是sorted set的元素数量

~~~java
GEOADD key longitude latitude member [longitude latitude member …]
GEOADD china 13.361389 38.115556 "shanghai" 15.087269 37.502669 "beijing"
~~~

**GEODIST**：计算指定的两个点之间的距离并返回

- 如果两个位置之间的其中⼀个不存在， 那么命令返回空值。
- **默认使用米**作为单位。
- GEODIST 命令在计算距离时会假设地球为完美的球形， 在极限情况下， 这⼀假设最⼤会造成 0.5% 的误差
- 返回值：计算出的距离会以双精度浮点数的形式被返回。 如果给定的位置元素不存在， 那么命令返回空值

~~~txt
GEODIST key member1 member2 [m|km|ft|mi]
GEODIST china beijing shanghai km
~~~

**GEOPOS**：返回指定member的坐标

- 返回值：GEOPOS 命令返回一个数组， 数组中的每个项都由两个元素组成： 第一个元素为给定位置元素的经度， 而第二个元素则为给定位置元素的纬度。当给定的位置元素不存在时， 对应的数组项为空值
- 复杂度：O(log(N)) 

~~~java
GEOPOS key member [member …]
geopos china beijing shanghai
~~~

**GEOHASH**：将指定member的坐标转化为hash字符串形式并返回，可以节省空间。

复杂度：O(log(N)) 

~~~java
GEOHASH key member [member …]
GEOHASH china beijing shanghai
~~~



**GEOSEARCH**：在指定范围内搜索member，并按照与制定点之间的距离排序后返回，范围可以使圆形或矩形

返回值：

- 在没有给定任何 WITH 选项的情况下， 命令只会返回一个像 [“New York”,”Milan”,”Paris”] 这样的线性（linear）列表。
- 在指定了 WITHCOORD 、 WITHDIST 、 WITHHASH 等选项的情况下， 命令返回一个二层嵌套数组， 内层的每个子数组就表示一个元素。
- 在返回嵌套数组时， 子数组的第一个元素总是位置元素的名字。 至于额外的信息， 则会作为子数组的后续元素， 按照以下顺序被返回：
  - 以浮点数格式返回的中心与位置元素之间的距离， 单位与用户指定范围时的单位一致。
  - geohash 整数。
  - 由两个元素组成的坐标，分别为经度和纬度

复杂度：

![image-20241026172353878](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241026172354.png)

~~~java
GEOSEARCH key [FROMMEMBER member] [FROMLONLAT longitude latitude] [BYRADIUS radius m|km|ft|mi] 
[BYBOX width height m|km|ft|mi] [ASC|DESC] [COUNT count [ANY]] [WITHCOORD] [WITHDIST] [WITHHASH]
    
geosearch china FROMLONLAT 15 37 BYRADIUS 200 km ASC WITHCOORD WITHDIST
~~~

WITHCOORD:返回结果时会附加每个结果的坐标。

WITHDIST: 如果包含此选项，返回结果时会附加每个结果与查询点的距离。

### 预热数据

把同类型的放入redis中，方便之后按照距离从redis中查询。

~~~java
 public void loadShopData(){
        //1.查询店铺信息
        List<Shop> list = shopService.list();
        //2.按照店铺id进行分组
        Map<Long, List<Shop>> mp = list.stream().collect(Collectors.groupingBy(Shop::getTypeId));
        //3.分批写入redis
        for (Map.Entry<Long, List<Shop>> entry : mp.entrySet()) {
            //3.1获取id
            Long typeId = entry.getKey();
            String key = "shop:geo:" + typeId;
            //3.2获取同类型的店铺集合
            List<Shop> shopList = entry.getValue();
            List<RedisGeoCommands.GeoLocation<String>> locations = new ArrayList<>(shopList.size());
            for (Shop shop : shopList) {
                //stringRedisTemplate.opsForGeo().add(key, new Point(shop.getX(), shop.getY()), shop.getId().toString());
                locations.add(new RedisGeoCommands.GeoLocation<>(
                        shop.getId().toString(), new Point(shop.getX(), shop.getY())));
            }
            stringRedisTemplate.opsForGeo().add(key, locations);
        }
    }
~~~

###  查询

~~~java
@Override
    public Result queryShopByType(Integer typeId, Integer current, Double x, Double y) {
        //1.判断需不需要查询距离
        if(x == null || y == null){
            Page<Shop> page = query()
                    .eq("type_id", typeId)
                    .page(new Page<>(current, SystemConstants.DEFAULT_PAGE_SIZE));
            return Result.ok(page);
        }
        //2.计算分页参数
        int from = (current - 1) * SystemConstants.DEFAULT_PAGE_SIZE;
        int end = current * SystemConstants.DEFAULT_PAGE_SIZE;
        //3.redis做分页查询,结果里面是店铺id和距离
        String key = "shop:geo:" + typeId;
        //这里查询的是0到end
        GeoResults<RedisGeoCommands.GeoLocation<String>> results = stringRedisTemplate.opsForGeo().search(
                key,
                GeoReference.fromCoordinate(x, y),
                new Distance(5000),
                RedisGeoCommands.GeoSearchCommandArgs.newGeoSearchArgs().includeDistance().limit(end)
        );
        //4.解析出id
        if(results == null){
            return Result.ok(Collections.emptyList());
        }
        List<GeoResult<RedisGeoCommands.GeoLocation<String>>> list = results.getContent();
        if(list.size() <= from){
            //没有下一页
            return Result.ok(Collections.emptyList());
        }
        //4.1截取from-end；
        List<Long> ids = new ArrayList<>(list.size());
        Map<String, Distance> distanceMap = new HashMap<>(list.size());
        list.stream().skip(from).forEach(result -> {
            //4.2获取店铺id
            String shopIdStr = result.getContent().getName();
            ids.add(Long.valueOf(shopIdStr));
            //4.3获取店铺距离
            Distance distance = result.getDistance();
            distanceMap.put(shopIdStr, distance);
        });
        //5根据id去查询数据库
        String idStr = StrUtil.join(",", ids);
        List<Shop> shops = query()
                .in("id", ids)
                .last("order by field ( id, " + idStr + ")")
                .list();
        for (Shop shop : shops) {
            shop.setDistance(distanceMap.get(shop.getId().toString()).getValue());
        }
        return Result.ok(shops);
    }
~~~

## 用户签到

我们针对签到功能完全可以通过MySQL来完成，例如下面这张表

用户签到一次，就是一条记录，假如有1000W用户，平均没人每年签到10次，那这张表一年的数据量就有1亿条

|   Field   |       Type       | Collation | Null | Key  | Default |     Extra      |  Comment   |
| :-------: | :--------------: | :-------: | :--: | :--: | :-----: | :------------: | :--------: |
|    id     | bigint unsigned  |  (NULL)   |  NO  | PRI  | (NULL)  | auto_increment |    主键    |
|  user_id  | bigint unsigned  |  (NULL)   |  NO  |      | (NULL)  |                |   用户id   |
|   year    |       year       |  (NULL)   |  NO  |      | (NULL)  |                |  签到的年  |
|   month   |     tinyint      |  (NULL)   |  NO  |      | (NULL)  |                |  签到的月  |
|   date    |       date       |  (NULL)   |  NO  |      | (NULL)  |                | 签到的日期 |
| is_backup | tinyint unsigned |  (NULL)   | YES  |      | (NULL)  |                |  是否补签  |

如何简化？可以用二进制存，第x位是1，表示第x+1天签到了。

Redis中是利用**String类型数据结构**实现BitMap，因此最大上限是512M，转换为bit则是2^32个bit位

用用户结合年和月作为key，值是当月的签到情况。

![image-20241028090444408](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241028090445.png)



- SETBIT：向指定位置（offset）存入一个0或1
- GETBIT：获取指定位置（offset）的bit值
- BITCOUNT：统计BitMap中值为1的bit位的数量
- BITFIELD：操作（查询、修改、自增）BitMap中bit数组中的指定位置（offset）的值，常用于**批量查询**，返回的是十进制

~~~txt
//从第0号位开始，拿dayOfMonth个
BITFIELD key GET u[dayOfMonth] 0
~~~

- BITFIELD_RO：获取BitMap中bit数组，并以十进制形式返回
- BITOP：将多个BitMap的结果做位运算（与、或、异或）
- BITPOS：查找bit数组中指定范围内第一个0或1出现的位置

~~~java
  @Override
    public Result sign() {
        //1.获取当前用户
        Long userId = UserHolder.getUser().getId();
        //2.获取日期
        LocalDateTime now = LocalDateTime.now();
        String formatTime = now.format(DateTimeFormatter.ofPattern(":yyyy/MM"));
        //3.拼接key
        String key = USER_SIGN_KEY + userId + formatTime;
        //4.获取本月是第几天
        int dayOfMonth = now.getDayOfMonth() - 1;
        //5.存入redis
        stringRedisTemplate.opsForValue().setBit(key, dayOfMonth, true);
        return Result.ok();
    }
~~~

连续签到统计：

~~~java
  public Result signCount() {
        //1.获取当前用户
        Long userId = UserHolder.getUser().getId();
        //2.获取日期
        LocalDateTime now = LocalDateTime.now();
        String formatTime = now.format(DateTimeFormatter.ofPattern(":yyyy/MM"));
        //3.拼接key
        String key = USER_SIGN_KEY + userId + formatTime;
        //4.获取本月是第几天
        int dayOfMonth = now.getDayOfMonth();
        //5.获取本月全部记录 BITFIELD sign:5:202203 GET U14 0
        List<Long> result = stringRedisTemplate.opsForValue().bitField(
                key,
                BitFieldSubCommands.create()
                        .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth))
                        .valueAt(0));
        if(result == null || result.size() == 0){
            return Result.ok(0);
        }
        //6.循环遍历
        Long num = result.get(0);
        int cnt = 0;
        while(true){
            if((num & 1) == 1){
                cnt ++;
            }
            else{
                break;
            }
            num >>>= 1;
        }
        return Result.ok(cnt);
    }
~~~

## UV统计

UV：全称Unique Visitor，也叫独立访客量，是指通过互联网访问、浏览这个网页的自然人。1天内同一个用户多次访问该网站，只记录1次。

PV：全称Page View，也叫页面访问量或点击量，用户每访问网站的一个页面，记录1次PV，用户多次打开页面，则记录多次PV。往往用来衡量网站的流量。

​	UV统计在服务端做会很麻烦，因为要判断该用户是否已经统计过了，需要将统计过的信息保存，但是如果每个访问的用户都保存到Redis中，那么数据库会非常恐怖，那么该如何处理呢？

​	HyperLogLog(HLL)是从Loglog算法派生的概率算法，用户确定非常大的集合基数，而**不需要存储其所有值**，而且多次插入一样的值**只插入一次**，因为天生适合存储UV。算法相关原理可以参考下面这篇文章：https://juejin.cn/post/6844903785744056333#heading-0

​	Redis中的HLL是基于**string结构实现的**，单个HLL的内存`永远小于16kb`，`内存占用低`的令人发指！作为代价，其测量结果是概率性的，`有小于0.81％的误差`。不过对于UV统计来说，这完全可以忽略。

~~~txt
PFADD key element [element...]
summary: Adds the specified elements to the specified HyperLogLog

PFCOUNT key [key ...]
Return the approximated cardinality of the set(s) observed by the HyperLogLog at key(s).
~~~

# Redis高级

## 分布式缓存

单点redis缺点：

- 数据丢失问题：Redis是内存存储，服务重启可能会丢失数据
- 并发能力问题：单节点Redis并发能力虽然不错，但也无法满足如618这样的高并发场景
- 故障恢复问题：如果Redis宕机，则服务不可用，需要一种自动的故障恢复手段
- 存储能力问题：Redis基于内存，单节点能存储的数据量难以满足海量数据需求

![image-20241101144912775](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241101144913.png)

## Redis的持久化

### 1.1RDB持久化

- RDB全称Redis Database Backup file（Redis数据备份文件），也被叫做Redis数据快照。
- 简单来说就是把内存中的**所有数据**都记录到磁盘中，不是只写更新的数据。
- 当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。
- 快照文件称为RDB文件，默认是保存在当前运行目录。

#### 1.1.1执行时机

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

#### 1.1.2.RDB原理

bgsave开始时会fork主进程得到子进程，子进程共享主进程的内存数据。完成fork后读取内存数据并写入 RDB 文件。

fork采用的是copy-on-write技术：

- 当主进程执行读操作时，访问共享内存；
- 当主进程执行写操作时，则会拷贝一份数据，执行写操作，极限情况内存数据会翻倍。

![image-20210725151319695](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102154252.png)

#### 1.1.3.小结

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

### 1.2.AOF持久化

#### 1.2.1.AOF原理

AOF全称为Append Only File（追加文件）。Redis处理的每一个写命令都会记录在AOF文件，可以看做是**命令日志文件**。

![image-20210725151543640](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102162722.png)



#### 1.2.2.AOF配置

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

#### 1.2.3.AOF文件重写

因为是记录命令，**AOF文件会比RDB文件大的多**。而且AOF会记录对同一个key的多次写操作，但**只有最后一次写操作才有意义**。通过执行bgrewriteaof命令，可以让AOF文件执行重写功能，用最少的命令达到相同效果。

![image-20210725151729118](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102163027.png)

Redis也会在触发阈值时自动去重写AOF文件。阈值也可以在redis.conf中配置：

```properties
# AOF文件比上次文件 增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# AOF文件体积最小多大以上才触发重写 
auto-aof-rewrite-min-size 64mb 
```

### 1.3.RDB与AOF对比

RDB和AOF各有自己的优缺点，如果对数据安全性要求较高，在实际开发中往往会**结合**两者来使用。

![image-20210725151940515](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/20241102163103.png)

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

### Redis主从
