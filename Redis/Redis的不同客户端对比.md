Redis 官方推荐的 Java 客户端有`Jedis`、`lettuce` 和 `Redisson`

| 客户端   | 简介                        | 优点                                                         | 缺点                      |
| -------- | --------------------------- | ------------------------------------------------------------ | ------------------------- |
| Jedis    | 提供了比较全面的 Redis 操作 | Jedis 简单全面, 支持 pipeline、事务等Redis高级特性           | `非线程安全的`不支持异步  |
| Lettuce  | 线程安全                    | 多个线程就可以共享一个连接，性能高                           | 学习使用成本高            |
| Redisson | 提供很多分布式相关操作      | 实现了分布式特性和可扩展的 Java 数据结构<br />线程安全且性能好 | 不支持Redis的一些高级特性 |

#### Jedis

Jedis 提供了比较全面的 Redis 命令的操作，也是目前使用最广泛的客户端。https://github.com/redis/jedis

**优点如下：**

- Jedis 的 API 支持比较全面的 Redis 命令,Java 方法基本和 Redis 的 API 保持着一致，容易上手
- 支持 pipeline、事务、Redis Sentinel、Redis Cluster等redis 提供的高级特性
- 客户端轻量，简洁，便于集成和改造

**缺点如下：**

- 使用阻塞的 I/O 操作，不支持异步
- Jedis 在多线程环境下是非线程安全的，使用连接池以解决 Jedis 客户端实例存在的[线程安全的问题](https://cloud.tencent.com/developer/article/1678172)
- 不支持读写分离，需要自己实现

整体来说，Jedis 是一款经典的 Redis 客户端（java 语言方向），能满足绝大部分项目中的业务开发需求，虽然有些瑕疵，但是可以通过其它方式来弥补，可用性、安全性方面都有保证，总体评价是操作简单，易上手！

#### Lettuce

Lettuce 是一种可扩展的、线程安全的 Redis 高级客户端。

从 Spring Boot 2.x 开始， Lettuce 已取代 Jedis 成为SpringBoot 默认的 Redis 客户端https://lettuce.io/

**优点如下：**

- 相比于 Jedis，Lettuce 属于后起之秀，对 Redis 更加全面，并且解决了 Jedis 客户端实例存在线程安全的问题
- 支持同步编程，异步编程，响应式编程，自动重新连接，主从模式，集群模块，哨兵模式，管道和编码器等等高级的 Redis 特性
- Lettuce 底层基于 Netty 框架的事件驱动与 redis 通信，采用了非阻塞的 I/O 操作，可异步调用，相比 Jedis，性能高
- Lettuce 的 API 是线程安全的，如果不是执行阻塞和事务操作，如 BLPOP 和MULTI/EXEC 等命令，多个线程就可以共享一个连接，性能方面不会衰减

**缺点如下：**

- API 学习使用成本高

#### Redisson

Redisson 是一个在 Redis 的功能基础上实现的客户端。实现了分布式和可扩展的 Java 数据结构，提供很多分布式相关操作服务，例如分布式锁，分布式集合，可通过 Redis 支持延迟队列。https://github.com/redisson/redisson

**优点如下：**

- 实现了分布式特性和可扩展的 Java 数据结构，例如分布式锁，分布式集合，分布式对象，分布式远程调度等等高级功能，适合分布式开发
- 与 Lettuce 一样，基于 Netty 框架的事件驱动与 redis 通信，支持异步调用，性能高
- Redisson 的 API 是线程安全的，所以可以使用单个 Redisson 连接来完成各种操作。
- 支持读写分离，支持读负载均衡，在主从复制和 Redis Cluster 架构下都可以使用
- 内建 Tomcat Session Manager，为 Tomcat 6/7/8 提供了会话共享功能，可以与 Spring Session 集成，实现基于 Redis 的会话共享
- 相比于 Jedis、Lettuce 等基于 redis 命令封装的客户端，Redisson 提供的功能更加高端和抽象，Redisson 可以类比 Spring 框架，这些框架搭建了应用程序的基础框架和功能，可以显著提升开发效率，让开发者有更多的时间来关注业务逻辑
- 文档较丰富，有中文文档

**缺点如下：**

- 和 Jedis、Lettuce 客户端相比，功能较为简单，对字符串的支持比较差，不支持排序、事务、管道、分区等 Redis 特性
- API 更加抽象，学习使用成本高

### 小结

Jedis 和 Lettuce 是比较纯粹的 Redis 命令客户端，几乎没提供什么分布式操作服务。

Jedis 和 Lettuce 两者相比，Jedis 的性能比较差，其他方面并没有太明显的区别，所以如果你不需要使用 Redis 的高级功能的话，优先推荐使用 Lettuce。

相比于 Jedis、Lettuce 等基于 redis 命令封装的客户端，Redisson 提供的功能更加高端和抽象

Redisson 的优势是提供了很多开箱即用的 Redis 高级功能，如果你的应用中需要使用到 Redis 的高级功能，比如分布式锁，分布式对象，分布式会话共享等等，建议使用 Redisson。

- 如果项目中对分布式功能的需求场景不多，优先推荐使用 Lettuce，基本上够用，使用 Jedis 也没问题，api 操作方面会更加简单。
- 如果项目中除了对基本的数据缓存操作以外，还需要用到分布式锁，分布式对象，分布式集合等功能，优先推荐采用`Lettuce` +`Redisson`组合方式使用。

### 扩展

#### jedis的线程安全问题

在多线程共用同一个jedis实例时,会存在一些资源被其他线程重置导致当前线程无法获取到资源,从而出现程序异常;参考https://cloud.tencent.com/developer/article/1678172

[参考原文链接](https://www.51cto.com/article/744307.html)