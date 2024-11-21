## 内容大纲

* 预备知识
* 5种数据结构的特点、命令使用、应用场景
* 键管理、遍历键、数据库管理



## 预备知识

### 常用命令

| 命令                | 用法（无法自解释的命令会加上相应的说明）                     |
| ------------------- | ------------------------------------------------------------ |
| keys *              | 查看所有键                                                   |
| dbsize              | 键总数                                                       |
| exists key          | 检查键是否存在                                               |
| del key             | 删除键                                                       |
| expire key seconds  | 设置键过期，键过期，会自动删除键                             |
| ttl key             | 观察键的过期时间（>0:键剩余的过期时间；-1:键没设置过期时间；-2:间不存在） |
| type key            | 键的类型                                                     |
| object encoding key | 查询内部编码                                                 |



### 数据结构和内部编码

Redis共有5种类型的数据结构：

* string
* hash
* list
* set
* zset

每种类型的数据结构都有其底层的实现方式：

<img src="%E5%85%A5%E9%97%A8.assets/image-20200824113810155.png" alt="image-20200824113810155" style="zoom: 33%;" />

Redis会根据配置选项在不同的情况下选择不同的内部实现。



### Redis的单线程架构

Redis架构采用**单线程架构和I/O多路复用模型**来实现高性能的内存数据库服务。单线程架构可用从2个方面去了解：

1. 单线程模型的机制
2. 单线程模型是如何达到如此高的性能的



**1.单线程模型的机制**

一条命令到达服务端后不会马上被执行，而是进入一个队列，然后Redis会使用一条单线程，从队列中一个一个的取出命令进行执行，所以只有一条执行命令线程的Redis是不会产生并发问题的。如下图所示：

![image-20220614200708494](常用数据结构.assets/image-20220614200708494.png)

**2.单线程模型的高性能**

高性能的原因可以归结为以下三点：

* 纯内存访问，内存相应时长大约为100纳秒
* 非阻塞IO
* 单线程避免了线程切换和竞态产生的消耗





## String（字符串）

键是字符串类型。值可以是：

* 字符串
* 数字
* 二进制

Max（值） = 512MB



### 命令

**常用**

| 命令        | 用法（无法自解释的命令会加上相应的说明）                     |
| :---------- | :----------------------------------------------------------- |
| INCR        | `INCR key`，将键存储的值+1。值不是整数，返回错误；是整数，返回自增后的结果；键不存在：按照值为0自增，返回结果为1 |
| DECR        | `DECR key`，将键存储的值-1                                   |
| INCRBY      | `INCRBY key amount`，将键存储的值+整数amount                 |
| INCRBYFLOAT | `INCRBYFLOAT key amount`，将键存储的值+浮点数amount          |
| APPEND      | `APPEND key value`在键存储的值的末尾追加一段字符串           |
| GETRANGE    | `GETRANGE key start end` ，获取start至end间的所有字符组成的子串，包括start和end |
| SET         | `SET KEY value [ex seconds][px milliseconds] [nx] [xx]`<br />参数含义：<br />ex seconds:为键设置秒级过期时间<br />px milliseconds:为键设置毫秒级过期时间。<br />nx:键必须不存在，才可以设置成功，用于添加<br />xx:键必须存在，才可以设置成功，用于更新 |
| MSET        | `MSET key value [key value ...]`，批量set值，批量set值性能更高 |
| MGET        | `MGET key [key ...]`，批量获取值，批量get值性能更高          |
| GETSET      | `GETSET key value`，返回原值，设置新值                       |

**不常用**

| 命令     | 用法（无法自解释的命令会加上相应的说明）                     |
| -------- | ------------------------------------------------------------ |
| STRLEN   | `STRLEN key`，返回字符串长度                                 |
| GETRANGE | `GETRANGE key start end` ，获取start至end间的（包括start和end）所有字符组成的子串 |
| APPEND   | `APPEND key value`在键存储的值的末尾追加一段字符串           |
| SETRANGE | `SETRANGE key offset value`，将从start开始的（包括start），长度为value的长度的子串设置为value |
| GETBIT   | `GETBIT key offset`，把键存储的字节串看成是二进制位串，并获取offset位置上的二进制位的值 |
| SETBIT   | `SETBIT key offset value`，把键存储的字节串看成是二进制位串，并把offset位置上的二进制位的值设置成value |
| BITCOUNT | `BITCOUNT key [start end]`，统计二进制位串中值为1二进制位的数量，start和end为可选参数，若给出这两个值，则只统计在start和end之间的二进制位串，统计时包含start和end处的值 |
| BITOP    | `BITOP opration dest-key key-name [key-name ...]`，对一个或多个二进制位串执行位操作，包括`AND、OR、XOR、NOT`在内的任意一种位操作，将处理后的结果保存在dest-key中 |



### 内部编码实现

* int：8个字节的长整型
* embstr：<=39个字节的字符串
* raw:>39个字节的字符串

Redis自己根据情况选择合适的内部编码实现。



### 典型使用场景

**1.缓存**

<img src="API的理解和使用.assets/image-20220614204516945.png" alt="image-20220614204516945" style="zoom:70%;" />

这种Redis+Mysql的缓存架构都烂大街了，没啥可说的，有些高并发场景下的注意事项，之后再记录。

> Redis键的设计技巧：
>
> 例如Mysql的数据库名为vs，表名为 user，那么对应的键可以采用:
>
> vs:user:{uid}:name

**2.计数**

```
long incrVideoCounter(long id) {
	key = "video:playCount:" + id; return redis.incr(key);
}
```

**3.共享Session**

<img src="API的理解和使用.assets/image-20220614213448636.png" alt="image-20220614213448636" style="zoom:50%;" />



**4.限速**

例如：手机验证码、IP地址不能再1s之内访问超过n次。

```
phoneNum = "138xxxxxxxx";
key = "shortMsg:limit:" + phoneNum;
// SET key value EX 60 NX
isExists = redis.set(key,1,"EX 60","NX"); if(isExists != null || redis.incr(key) <=5){
	// 通过 
}else{
	// 限速 
}
```



## Hash（哈希）

<img src="API的理解和使用.assets/image-20220614213908307.png" alt="image-20220614213908307" style="zoom:70%;" />



### 命令

| 命令          | 用法（无法自解释的命令会加上相应的说明）                     |
| ------------- | ------------------------------------------------------------ |
| HMGET         | `HMGET key-name key [key ...]`                               |
| HMSET         | `HMSET key-name key value [key value]`                       |
| HDEL          | `HDEL key-name key [key ...]`                                |
| HLEN          | `HLEN key-name`                                              |
| HEXISTS       | `HEXISTS key-name key`                                       |
| HKEYS         | `HKEYS key-name`                                             |
| HVALS         | `HVALS key-name`                                             |
| HGETALL       | `HGETALL key-name`，哈希元素个数比较多的时候，可能会产生阻塞 |
| HINCRBY       | `HINCRBY key-name key value`，将key存储的值加上整数value     |
| HINCRBY FLOAT | `HINCRBY FLOAT key-name key value`，将key存储的值加上浮点数value |
| HSTRLEN       | `HSTRLEN key-name key`，计算key存储的值的长度                |
| HSET          | `HSET key filed value `                                      |
| HGET          | `HGET key field`                                             |



### 内部编码

* ziplist

  field-value个数<512个 && 所有value值<64字节，采用此结构。

  可用`hash-max-ziplist-entries、hash-max-ziplist-value`进行配置。

* hashtable

  不满足ziplist的条件，采用此结构



### 使用场景

**保存用户信息**

<img src="API的理解和使用.assets/image-20220614215734984.png" alt="image-20220614215734984" style="zoom:70%;" />

一个key代表MySQL表中的一行数据。

优点：简单直观

缺点：要控制哈希在ziplist和hashtable两种内部编码的转换，hashtable会消耗更多内存。



## 列表

* 用来存储多个**有序**的字符串；

* 最多可存储2^32 -1个元素；
* 可充当栈、队列的角色。

* 有序
* 可重复
* 支持的操作：
  * 增、删、改、查
  * 阻塞操作

### 命令

| 命令       | 用法（无法自解释的命令会加上相应的说明）                     |
| ---------- | ------------------------------------------------------------ |
| RPUSH      | `RPUSH key value [value ...]`                                |
| LPUSH      | `LPUSH key value [value ...] `                               |
| RPOP       | `RPOP key`                                                   |
| LPOP       | `LPOP key`                                                   |
| LINDEX     | `LINDEX key offset`                                          |
| LRANGE     | `LRANGE key start end`                                       |
| LTRIM      | `LTRIM key start end`，对列表进行修剪，保留list中索引在start和end之间的值（包括start和end） |
| BLPOP      | `BLPOP key [key ...] timeout`，从第一个非空列表中弹出位于最左端的元素，或者在timeout秒内阻塞并等待可弹出的元素出现 |
| BRPOP      | `BLPOP key [key ...] timeout`，从第一个非空列表中弹出位于最右端的元素，或者在timeout秒内阻塞并等待可弹出的元素出现 |
| RPOPLPUSH  | `RPOPLPUSH source-key des-key`，从source-key列表中弹出最右侧的元素，并推入des-key列表的最左端，并向用户返回这个元素 |
| BRPOPLPUSH | `BRPOPLPUSH source-key des-key timeout`，从source-key列表中弹出最右侧的元素，并推入des-key列表的最左端，并向用户返回这个元素；如果source-key列表为空，那么在timeout秒之内等待可弹出元素的出现 |



### 内部编码

* ziplist

  列表元素个数<512个 && 每个元素的值都<64字节，使用此编码。

  可用`list-max-ziplist-entries、list-max-ziplist-value`进行配置。

* linkedlist

  不满足ziplist的条件，使用此编码。

* quicklist

  Redis3.2版本提供的实现。结合了ziplist和linkedlist两者的优势。



### 使用场景

* lpush+lpop=Stack(栈) 
* lpush+rpop=Queue(队列) 
* lpsh+ltrim=Capped Collection(有限集合) 
* lpush+brpop=Message Queue(消息队列)



## Set（集合）

* 不重复
* 无序
* 不能通过索引下标获取元素
* 最多存储2^32-1个元素
* 支持的操作
  * 增、删、改、查
  * 交、并、差



### 命令

| 命令        | 用法（无法自解释的命令会加上相应的说明）                     |
| ----------- | ------------------------------------------------------------ |
| SADD        | `SADD key item [item ...]`                                   |
| SREM        | `SREM key item [item ...]`                                   |
| SISMEMBER   | `SISMEMBER key item`                                         |
| SCARD       | `SISMEMBER key`，返回集合中元素的数量                        |
| SMEMBERS    | `SMEMBER key`，返回集合中所有的元素                          |
| SRANDMEMBER | `SRANDMEMBER key [count] `，随机的返回集合中count绝对值个元素，count为负值返回的元素可重复，count为正值返回的元素不会重复 |
| SPOP        | `SPOP key`，随机的移除集合中的一个元素，并将这个元素返回     |
| SMOVE       | `SMOVE source-key des-key item`，如果集合source-key中包含元素item，会把item从结合source-key中移除，并添加到des-key中，返回1；如果集合source-key中不包含item，返回0。 |
| SDIFF       | `SDIFF key [key ...]`，返回那些存在于第一个集合但不存在于其他集合的元素，数学差集运算 |
| SINTER      | `SINTER key [key ...]`，返回那些存在于所有集合中的元素，数学交集运算 |
| SINTERSTORE | `SINTERSTORE des-key key [key ...]`，将那些存在于所有集合中的元素（数学交集运算）存放到des-key中 |
| SUNION      | `SUNION key [key ...]`，返回那些至少存在于一个集合中的元素，数据并集运算 |
| SUNIONSTORE | `SUNIONSTORE des-key key [key ...]`，将那些至少存在于一个集合中的元素（数据并集运算）存放到des-key中 |



### 内部编码

* intset

  当集合中的元素都是整数 && 元素个数小于512个，使用此编码

  可以通过`set-max- intset-entries`进行配置

* hashtable



### 使用场景

比较典型的使用场景是：标签。





## Zset（有序集合）

保留了Set的特性，但是可排序。

![image-20220615164818515](API的理解和使用.assets/image-20220615164818515.png)



### 命令

| 命令             | 用法（无法自解释的命令会加上相应的说明）                     |
| ---------------- | ------------------------------------------------------------ |
| ZADD             | `ZADD key score member [score member]`                       |
| ZREM             | `ZREM key member [member ...]`                               |
| ZCARD            | `ZCARD key`，返回有序集合包含的元素数量                      |
| ZINCRBY          | `ZINCRBY key increment member`，将member的分值加上increment  |
| ZCOUNT           | `ZCOUNT key min max`，返回分值介于min和max之间的成员的数量   |
| ZRANK            | `ZRANK key member`，返回member在集合中的排名                 |
| ZSCORE           | `ZSCORE key member`，返回member在集合中的分值                |
| ZRANGE           | `ZRANGE key start stop [WITHSCORES]`，返回排名介于start和stop之间的成员，如果给定了可选参数`WITHSCORES`，则会将分值一起返回 |
| ZREVRANK         | `ZREVRANK key member`，返回有序集合中成员member的排名，成员按照分值从大到小排列 |
| ZREVRANGE        | `ZREVRANGE key start stop [WITHSCORES]`，返回有序集合中**排名**在start到stop之间的成员，成员按照分值从大到小排列 |
| ZRANGEBYSCORE    | `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]` ，返回有序集合中，**分值**在min和max之间的所有成员 |
| ZREVRANGEBYSCORE | `ZREVRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]` ，返回有序集合中，分值在min和max之间的所有成员，并按照**分值**从大到小的顺序返回他们 |
| ZINTERSTORE      | `ZINTERSTORE dest-key key-count key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]`，交集运算 |
| ZUNIONSTORE      | `ZINTERSTORE dest-key key-count key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM`，并集运算 |
| ZREMRANGEBYRANK  | `ZREMRANGEBYRANK key start stop`，移除有序集合中**排名**介于start和stop之间的所有成员 |
| ZREMRANGEBYSCORE | `ZREMRANGEBYSCORE key min max`，移除有序集合中分值介于min和start之间的所有成员 |



### 内部编码

* ziplist

  集合中的元素个数小于128个 && 每个元素的值都小于64字节

  可以通过`zset-max-ziplist- entries、zset-max-ziplist-value`进行配置

* skiplist



### 使用场景

* 添加用户赞数 
* 取消用户赞数
* 展示获取赞数最多的10个用户
* 展示用户信息以及用户分数



## Bitmaps

* Bitmaps本身不是一种数据结构，实际上它就是字符串，但是可以对字符串的位进行操作
* Bitmaps单独提供了一套命令

![image-20220615195825636](API的理解和使用.assets/image-20220615195825636.png)



### 命令



| 命令     | 用法（无法自解释的命令会加上相应的说明）              |
| -------- | ----------------------------------------------------- |
| setbit   | setbit key offset value                               |
| getbit   | getbit key offset                                     |
| bitcount | bitcount [start] [end],获取Bitmaps指定范围值为1的个数 |
|          |                                                       |











