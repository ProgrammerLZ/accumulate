> 对于每一项要学习的任务，必须先要了解这项技术在实际的业务系统当中，是为了解决哪些主要的实际问题。根据其能够解决的问题的大小，来对这项技术的重要程度进行初步的评判。这样也能够在以后的业务系统当中，当遇到了某些实际发生的问题的时候，能够对症下药，采用最有效、最合适的技术去解决实际的问题

### Java8

1. 行为参数化
2. 函数式数据处理
3. Optional
4. CompletableFuture：组合式 异步编程
5. 新的日期和时间API



### MQ

1. 体验一下面试官对于消息队列的7个连环炮
2. MQ的作用
3. MQ保证消息队列的高可用
4. MQ保证消息不被重复消费
5. MQ如何保证消息的顺序性
6. MQ如何处理消息丢失的问题
7. MQ消息持续积压几小时怎么处理



### Redis

1. Redis线程模型
2. Redis数据类型以及用的场景
3. Redis过期策略
4. Redis持久化RDB/AOF
5. Redis集群模式架构思想/gossip通信协议/hash slot算法/部署经验
6. Redis高可用架构
7. 高并发场景下的缓存一致性问题



### 分库分表

1. 体验一下面试官对于分库分表这个事儿的一个连环炮
2. 垂直拆分/水平拆分
3. 动态扩容/不停机迁移
4. 分库分表后的全局ID方案



### 熔断技术必看

1. 资源隔离/熔断/降级的作用
2. 线程池/信号量两种隔离级别技术理解
3. 熔断技术的执行步骤和流程



### Springcloud

1. Springcloud底层架构原理
2. Springcloud与Dubbo的对比
3. Springcloud的组件



### 注册中心

1. Eureka注册中心基础原理
2. Zookeeper注册中心基础原理
3. Euerka注册中心参数调优



### 分布式事务

1. 怎么设计分布式事务技术方案
2. TCC事务、最终一致性事务的技术选型
3. RocketMQ对分布式事务支持的底层实现原理



### 并发编程

1. HashMap底层结构/JDK1.8 HashMap的优化/hash算法/寻址算法
2. HashMap如何解决hash碰撞/如何扩容
3. Volatile/synchronized底层原理(原子性、可见性、顺序性)
4. CAS底层原理/CAS会引发的问题
5. AQS底层原理/AQS核心的执行流程
6. 线程池的底层工作原理
7. 线程池的核心配置参数
8. ConcurrentHashMap底层原理



### 网络部分

1. TCP/IP四层网络模型
2. 浏览器请求的全过程是怎么样的
3. HTTPS的工作原理



### Zookeeper

1. ZooKeeper可以做什么？
2. ZooKeeper集群的三种角色：Leader、Follower、Observer
3. ZooKeeper最核心的一个机制：Watcher监听回调
4. ZAB的核心思想介绍：主从同步机制和崩溃恢复机制
5. ZooKeeper到底是强一致性还是最终一致性



### JVM

1. 类加载机制/类加载过程
2. 双亲委派机制/以及他的作用
3. JVM内存区域(堆、栈等其他)
4. 堆内存划分
5. 新生代、老年代垃圾回收机制以及原理
6. 常用的垃圾回收器(ParNew、CMS、G1)
7. 触发垃圾回收的时机和条件
8. JVM调优参数模板
9. JVM分析命令(jstat、jmap、jstack等)
10. JVM分析工具(MAT等)



### Rocketmq

1. Rocketmq架构原理
2. Rocketmq场景优化
3. Rocketmq底层原理/消息持久化/raft协议
4. Rocketmq事务机制
5. Rocketmq消息0丢失
6. Rocketmq重复消息/死信队列



### Mysql

1. Mysql架构设计(SQL接口、查询解析器、查询优化器、存储引擎、执行器)
2. Mysql日志相关(undo log、redo log、binlog)
3. Innodb存储引擎(buffer pool结构、free链表作用、flush链表作用、lru链表作用)
4. Mysql数据页/数据行/表空间/数据区
5. Mysql事务隔离级别/MVCC机制原理
6. 索引原理/索引规则/索引优化
7. Explain执行计划分析



### 设计模式

1. 策略模式
2. 迭代器模式
3. 工厂模式
4. 代理模式
5. 构造器模式
6. 适配器模式



### 微服务组件

3. Ribbon负载均衡工作原理/负载均衡算法IRule
4. Hystrix资源隔离、限流、熔断、降级
5. Hystrix的线程池隔离技术/信号量隔离技术
6. Hystrix执行时的8大流程步骤以及内部原理



### 分布式事务

1. 事务的基础知识：ACID以及几种隔离级别
2. 事务的基础知识：Spring的事务支持以及传播特性
3. XA规范以及2PC分布式事务理论介绍
4. 2PC分布式事务方案的缺陷以及问题/3PC分布式事务方案的理论知识讲解
5. 理解CAP与BASE的基础知识
6. 理解TCC分布式事务技术方案以及原理/TCC分布式事务执行流程
7. 可靠消息最终一致性方案/整体架构以及核心流程设计



### 算法



### 分布式系统架构实战

1. 保证分布式系统的接口幂等性的几种常见方案
2. Redisson分布式锁原理
3. Redisson可重入锁/lua脚本加锁逻辑/watchdog维持加锁/锁的互斥阻塞/释放锁
4. Redisson公平锁原理
5. MultiLock原理/RedLock源码算法实现
6. Zookeeper Curator框架介绍以及分布式锁支持
7. Zookeeper分布式锁可重入锁源码剖析
8. Zookeeper分布式锁可重入读写锁源码剖析



### JDK源码剖析

#### 集合部分

1. ArrayList基本原理以及优缺点
2. ArrayList数组扩容以及元素拷贝
3. LinkedList基本原理以及优缺点
4. LinkedList双向链表数据结构
5. HashMap数组、链表、红黑树的数据结构
6. HashMap优化后的降低冲突概率的hash算法
7. HashMap put操作原理以及hash寻址算法
8. HashMap JDK 1.8引入红黑树优化hash冲突
9. HashMap JDK 1.8的高性能rehash算法
10. LinkedHashMap底层原理/有顺序的map数据结构
11. TreeMap/HashSet/LinkedHashSet/TreeSet底层结构以及原理
12. Iterator迭代器应对多线程并发修改的fail fast机制



#### 并发编程部分

1. CPU多级缓存模型
2. 总线加锁机制和MESI缓存一致性协议
3. Java内存模型
4. volatile是如何保证可见性/顺序性
5. volatile的底层实现原理：lock指令以及内存屏障
6. synchronized底层原理（jvm指令以及monitor锁）
7. 可见性涉及的底层硬件概念：寄存器、高速缓存、写缓冲器
8. 深入探秘有序性：Java程序运行过程中发生指令重排的几个地方
9. synchronized锁同时对原子性、可见性以及有序性的保证
10. 采用写缓冲器和无效队列优化MESI协议的实现性能
11. AtomicInteger中的CAS无锁化原理
12. AtomicInteger源码剖析：仅限JDK内部使用的Unsafe类
13. AtomicInteger源码剖析：底层CPU指令是如何实现CAS语义的
14. Atomic原子类体系的CAS语义存在的三大缺点分析/CAS的引发问题的解决方案
15. AQS的原理(异步队列同步器)/以及AQS重要的参数
16. AQS默认的非公平加锁策略的运作原理
17. AQS队列唤醒阻塞线程的过程
18. ReentractLock如何设置公平锁策略
19. ThreadLocal源码剖析：线程本地副本的实现原理
20. JDK 1.7 HashMap并发环境下死循环之环形链表
21. JDK 1.7 HashMap并发环境下死循环之死循环与丢数据
22. ConcurrentHashMap底层原理：初始化流程
23. ConcurrentHashMap底层原理：未分段数组的CAS加锁
24. ConcurrentHashMap底层原理：链表和红黑树解决hash冲突问题
25. JDK 1.8对ConcurrentHashMap做出的锁细粒度优化
26. CopyOnWriteArrayList：线程安全的List数据结构/基于写时复制机制
27. CopyOnWriteArrayList核心思想：弱一致性提升读并发
28. 线程安全的有界队列：LinkedBlockingQueue
29. 基于数组实现的有界队列：ArrayBlockingQueue
30. 线程池的核心成员变量有哪些/以及作用
31. 线程池的源码执行流程
32. 线程池的几个种类/区别/以及使用场景



### 网络

1. 数据链路层：以太网协议、mac地址、网卡以及路由器
2. 网络层：IP协议、子网划分以及子网掩码
3. 传输层：TCP协议、Socket编程是什么
4. 应用层：HTTP协议是什么
5. HTTPS协议加密通信的实现原理
6. 全世界几万台DNS服务器如何大接力完成IP地址的查询
7. ARP广播获取路由器mac地址以及ARP缓存机制



### 公司的项目

1. 梳理自己做过比较复杂的或者最有挑战性的功能或者项目
2. 把上面的功能或者项目画一下核心的流程图
3. 把上面的功能或者项目一些功能点列一下，还有表结构串联起来
4. 做完这些功能和项目难点在哪，从里面学到什么东西，自己回顾一下
5. 梳理项目的时候，参考闲人大佬之前在群里分享的内容，比如架构班学员如何梳理项目，按照他们的思路去梳理项目。

以上内容基本都是我面试前的准备，基本是我挑选过的一些重点知识。 有些课程之前看过，复习的时候过过笔记就行了，但大多时候，我会有重新看视频或者重新看专栏课的文章。



* 单机每秒 1600 条数据的处理

  两次架构升级，在MySQL前边加了一个Redis缓存，批处理去定期的处理Redis缓存中的数据；把Redis拿掉，换成了MQ，把批处理换成了一个消费者系统，专门用于处理MQ中的数据。

* 不规则图形融合算法的实现

  

* 站内搜索

* 配置文件+RocketMQ 的技术手段，对原用户行为追踪日志业务进行重构，减少了大量业务端代码的编写

  用Spring Boot的自动配置的功能做了一个日志追踪的自动配置的包，把这个包引入工程后，这个包会自动检测所在的工程是否存在某个配置文件，如果存在就会在Spring容器中自动注入Consumer的bean或者Producer的bean，具体注入什么样的bean取决于所在的工程被追踪的那个配置文件的名称叫什么。生产者工程会拦截用户请求，并将拦截到的请求截取有用的日志信息，并最终转换为MQ的消息发送到MQ中去。

* Java 集合类

* Spirng、SpringMVC、SpringBoot、MyBatis

* 熟练使用 RocketMq 并理解其原理

* NIO 的基本原理

* 多线程技术











