# Spring-boot集成Redis

## Redis简介

Redis是一个开源的使用 ANSI C语言编写，支持网络，可基于内存也可持久化的日志型，Key-Value数据库，并提供了多种语言的 API ,相比 `Memcached` 它支持存储的类型相对更多 (字符，哈希，集合，有序集合，列表等)，同时Redis是线程安全的。

## Redis Java Client

Redis Java Client 是面向Java的连接Redis Server的客户端，提供Java语言调用接口。目前主流的Redis客户端，是 Lettuce 和 Jedis。

- Jedis 在实现上是直连 redis server，多线程环境下非线程安全，除非使用连接池，为每个 redis实例增加物理连接。
- Lettuce 是 一种可伸缩，线程安全，完全非阻塞的Redis客户端，多个线程可以共享一个RedisConnection,它利用Netty NIO 框架来高效地管理多个连接，从而提供了异步和同步数据访问方式，用于构建非阻塞的反应性应用程序。

---

**注意**：在 springboot 1.5.x版本的默认的Redis客户端是 Jedis实现的，springboot 2.x版本中默认客户端是用 lettuce实现的。

## Spring Data Redis

Spring-data-redis是spring大家族的一部分，提供了在srping应用中通过简单的配置访问redis服务，对底层redis客户端(Lettuce 、Jedis)进行了高度封装，通过RedisTemplate对象为redis交互提供了高级抽象（主要负责连接管理和序列化工作），同时也通过RedisConnection对象提供接受和返回二进制值（字节数组）的低级方法。

RedisTemplate通过将Redis命令分组并抽象成操作视图（operations views），并通过操作视图的提供的接口来调用具体的redis命令，不同的操作视图对应了特定的Redis存储类型或者某些秘钥（keybound）。模板的操作视图具体如下：

| Interface              | Description                                               |
| ---------------------- | --------------------------------------------------------- |
| *Key Type Operations*  |                                                           |
| GeoOperations          | Redis geospatial operations like `GEOADD`, `GEORADIUS`,…) |
| HashOperations         | Redis hash operations                                     |
| HyperLogLogOperations  | Redis HyperLogLog operations like (`PFADD`, `PFCOUNT`,…)  |
| ListOperations         | Redis list operations                                     |
| SetOperations          | Redis set operations                                      |
| ValueOperations        | Redis string (or value) operations                        |
| ZSetOperations         | Redis zset (or sorted set) operations                     |
| *Key Bound Operations* |                                                           |
| BoundGeoOperations     | Redis key bound geospatial operations.                    |
| BoundHashOperations    | Redis hash key bound operations                           |
| BoundKeyOperations     | Redis key bound operations                                |
| BoundListOperations    | Redis list key bound operations                           |
| BoundSetOperations     | Redis set key bound operations                            |
| BoundValueOperations   | Redis string (or value) key bound operations              |
| BoundZSetOperations    | Redis zset (or sorted set) key bound operations           |

RedisTemplate可以通过调用其`opsFor[X]`方法来获得该模板具体的操作视图：

- **opsForValue：** 对应 String（字符串）
- **opsForZSet：** 对应 ZSet（有序集合）
- **opsForHash：** 对应 Hash（哈希）
- **opsForList：** 对应 List（列表）
- **opsForSet：** 对应 Set（集合）
- **opsForGeo：** 对应 GEO（地理位置）

## Redis 连接池简介

在spring-boot中使用连接池来管理redis连接，所以对连接池进行简单说明：

---

**客户端连接 Redis 使用的是 TCP协议，直连的方式每次需要建立 TCP连接，而连接池的方式是可以预先初始化好客户端连接，所以每次只需要从 连接池借用即可**，而借用和归还操作是在本地进行的，只有少量的并发同步开销，远远小于新建TCP连接的开销。另外，直连的方式无法限制 redis客户端对象的个数，在极端情况下可能会造成连接泄漏，而连接池的形式可以有效的保护和控制资源的使用。

<==To be continue...

## springboot 2.0使用lettuce集成Redis服务

### 依赖

```

```

### application.properties配置文件

### 自定义 RedisTemplate

### 封装Redis操作接口

### 参考

​	[http://www.spring4all.com/article/1152](http://www.spring4all.com/article/1152)

​	[https://juejin.im/post/5ba0a098f265da0adb30c684]([https://juejin.im/post/5ba0a098f265da0adb30c684)

​	[https://blog.csdn.net/weixin_42430194/article/details/80834733](https://blog.csdn.net/weixin_42430194/article/details/80834733)

​	[https://docs.spring.io/spring-data/redis/docs/2.0.1.RELEASE/reference/html/#why-spring-redis]([https://docs.spring.io/spring-data/redis/docs/2.0.1.RELEASE/reference/html/#why-spring-redis)

# 