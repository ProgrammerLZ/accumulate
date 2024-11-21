# Redis持久化

> Redis的持久化，主要是为了解决：因为进程退出造成的数据丢失问题，当下次重启时利用之前持久化的文件即可实现数据恢复。

Redis支持RDB和AOF两种持久化机制：

* RDB
* AOF



## RDB

> RDB持久化是把当前进程数据生成快照保存到硬盘的过程。

RDB持久化的触发方式分为两种：

* 手动触发
* 自动触发



### 触发方式

**手动触发**

| 命令   | 作用                                                         |
| ------ | ------------------------------------------------------------ |
| save   | 阻塞当前Redis服务器，知道RDB过程完成为止；内存 比较大的实例会造成长时间阻塞，线上环境不建议使用;**已废弃** |
| bgsave | Redis进程执行fork操作创建子进程，RDB持久化过程由子 进程负责，完成后自动结束 |



**自动触发**

* 使用save相关配置，如`save m n`。表示m秒内数据集存在n次修改 时，自动触发bgsave。
* 如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点
* 执行debug reload命令重新加载Redis时，也会自动触发save操作。
* 默认情况下执行shutdown命令时，如果没有开启AOF持久化功能则 自动执行bgsave。



### bgsave流程

<img src="Redis的持久化.assets/image-20220617231652788.png" alt="image-20220617231652788" style="zoom:50%;" />

* 通 过`info stats`命令查看`latest_fork_usec`选项，可以获取最近一个fork操作的耗 时，单位为微秒。
* 执行`lastsave`命令可以获取最后一次生成RDB的时间，对应info统计的`rdb_last_save_time`选项。



### RDB文件

**保存相关配置**

| 配置项     | 作用                  |
| ---------- | --------------------- |
| dir        | 配置RDB文件的保存目录 |
| dbfilename | 配置RDB的文件名       |

运行期间可通过：

* `config set dir {newDir}`
* `config set dbfilename {newFileName}`

动态进行配置。



**压缩相关配置**

默认采用LZF算法进行压缩处理，默认开始压缩配置。可通过`config setrdcompression{yes|no} `动态修改。



**文件校验**

若RDB文件损坏导致Redis无法正常启动，可采用Redis提供的redis-check-dump工具进行检测。



### RDB的优缺点

**优点**

* RDB文件是一个紧凑压缩的二进制文件，非常适用于备份、全量复制等场景。
* Redis加载RDB回复数据远快于AOF方式。



 **缺点**

* 无法做到实时持久化，因为每次运行都要执行fork操作创建子进程，属于重量级操作





## AOF

> AOF持久化是以独立日志的方式记录每次写命令，重启时再重新执行AOF文件命令达到恢复数据的目的。

AOF主要解决了Redis数据持久化的实时性问题，是Redis持久化的主流方式。



### 配置

| 配置项         | 含义                                |
| -------------- | ----------------------------------- |
| appendonly     | 是否开始AOF功能，默认不开启         |
| appendfilename | 配置AOF文件名，默认：appendonly.aof |

### 运行流程

<img src="Redis的持久化.assets/image-20220617233257388.png" alt="image-20220617233257388" style="zoom:50%;" />

#### append

append命令是把所有写入命令都追加到AOF缓冲当中。

* 命令写入直接是以纯文本的方式写入
* 命令会直接写入aof_buf中
  * 避免每次都写入硬盘导致的性能问题
  * 加一个缓冲区，相当于加了一个中间层，可以让Ridis有机会配置各种刷盘策略，从而在性能和安全性方面做出平衡



#### sync

sync操作是把AOF缓冲中的数据同步到AOF文件中，sync有多重同步策略，同步策略可由参数`appendfsync`进行控制。

| 可配置值 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| always   | 每次写入命令写入到aof_buf后，进行系统调用fnsyc，把数据同步至AOF文件。该方式对性能不友好，但是保证了数据的安全性 |
| everysec | 每次写入命令写入到aof_buf后，进行系统调用write。fsync同步文件操作由专门线程，每秒调用一次。 |
| no       | 每次写入命令写入到aof_buf后，进行系统调用write。不对AOF文件做fsync同步，同步硬盘操作由操作系统负责，通常同步周期最长30s。该方式对性能友好，但是安全性很低 |

> write和fsync
>
> * write：触发延迟写机制。数据会写入Linux内核缓冲区，之后会有操作系统调度机制来控制何时把这些数据同步进磁盘；
> * fsync：针对单个文件做强制硬盘同步。它会阻塞，知道写入硬盘完成后返回。



#### rewrite

rewrite主要是解决随着命令不断写入AOF导致文件越来越大的问题。它会压缩AOF文件的体积。**它是把Redis进程内的数据转化为写命令同步到新AOF文件的过程**。具体做法如下：

* 超时的数据不再写入文件；
* 移除旧AOF文件的无效命令；
* 多条命令合并为一个。

重写过程可以手动触发，亦可以自动触发。

手动触发：直接调用`bgrewriteof`命令

自动触发：涉及到两个参数，`auto-aof-rewrite-min-size`和`auto-aof-rewrite-percentage`。

| 配置参数                    | 作用                                                         |
| --------------------------- | ------------------------------------------------------------ |
| auto-aof-rewrite-min-size   | 表示运行AOF重写时文件最小体积，默认 为64MB。                 |
| auto-aof-rewrite-percentage | 代表当前AOF文件空间 (aof_current_size)和上一次重写后AOF文件空间(aof_base_size)的比 值。 |

当`aof_current_size > auto-aof-rewrite-min-size && (aof_current_size-aof_base_size)/aof_base_size>=auto-aof-rewrite-percentage`，会自动触发AOF重写。

AOF重写的具体流程也是比较重要的一个环节，其主要流程如图所示。

<img src="Redis的持久化.assets/image-20220618163642404.png" alt="image-20220618163642404" style="zoom:50%;" />

1. 如果AOF正在执行重写，返回错误；如果Redis正在执行bgsave操作，重写操作则进入Redis的调度，等bgsave完成后再执行重写操作；
2. 父进程fork创建子进程；
3. fork操作完成后，主进程中会存在两个写命令缓冲区：
   1. aof_buf（原缓冲区），作用跟之前所述一致；
   2. aof_rewrite_buf（重写缓冲区），保存新命令，子进程创建完新的AOF文件之后，会把这部分命令写入新AOF文件
4. 子进程根据内存快照，创建新AOF文件。此过程中，在主进程所有产生的新命令，子进程完全不知道，这也是为什么要有aof-rewirte_buf的原因。每次批量写入硬盘的数据量由`aof-rewrite-incremental-fsync`控制，默认32MB；
5. 新AOF文件写入完成
   1. 子进程发送信号给父进程，父进程更新统计信息，可通过info persistence下的aof_*进行查看；
   2. 父进程把AOF重写缓冲区的数据写入到新的AOF文件；
   3. 新替老



#### load

<img src="Redis的持久化.assets/image-20220618164758368.png" alt="image-20220618164758368" style="zoom:50%;" />

load过程涉及到加载AOF文件，AOF损坏的话Redis会拒绝启动，此时可以进行如下操作：

1. 备份AOF文件
2. 采用redis-check-aof --fix工具进行修复，该工具由Redis提供，到Redis的安装目录下寻找即可
3. 修复后使用diff -u命令对比数据差异，找出丢失数据，有些数据可以人工补全

另外，`aof-load-truncated`这项配置可以兼容AOF文件由于机器突然掉电导致的AOF尾部文件写入不全的问题，默认开启。



## 持久化问题定位与优化

持久化是影响Redis性能的问题高发地。



### fork操作

fork是一个重量级操作，对于高流量的Redis实例，OPS > 5万，若fork耗时在秒级，将拖慢Redis几万命令的执行。正常的fork耗时应该是每GB消耗20ms左右，可通过info stats统计中查latest_fork_usec指标来获取。



优化方法：

1. 优先使用物理机或者搞笑支持fork操作的虚拟化技术
2. 控制Redis实例最大可用内存——fork耗时跟内存量成正比，建议每个Redis实力内存<10GB
3. 合理配置Linux内存分配策略，避免物理内存不足导致fork失败
4. 降低fork操作的频率



### 子进程开销

子进程的作用主要是RDB文件的生成和AOF文件的重写，主要涉及CPU、内存、硬盘三部分的消耗。

1. CPU

开销：子进程负责把进程内的数据分批写入文件，通常子进程对单核CPU利用率接近90%。

优化：

* 不要做绑定单核CPU操作
* 不要和其他CPU密集型服务部署在一起
* 如果部署多个Redis实例，尽量保证同一时刻只有一个子进程执行重写工作

2. 内存

开销：重写过程中，存在内存修改操作，RDB中父进程创建所修改内存页的副本，而AOF父进程还需创建重写缓冲区

优化：

* 同CPU优化一样，如果部署多个Redis实例，尽量保证同一时刻只有 一个子进程在工作。
* 避免在大量写入时做子进程重写操作，这样将导致父进程维护大量页副本，造成内存消耗。

3. 硬盘

开销：子进程主要职责是把AOF或者RDB文件写入硬盘持久 化。势必造成硬盘写入压力

优化：

* 不要和其他高硬盘负载的服务部署在一起。如:存储服务、消息队列服务等。 

* AOF重写时会消耗大量硬盘IO，可以开启配置`no-appendfsync-on-rewrite`，默认关闭。表示在AOF重写期间不做fsync操作。
* 当开启AOF功能的Redis用于高流量写入场景时，如果使用普通机械 磁盘，写入吞吐一般在100MB/s左右，这时Redis实例的瓶颈主要在AOF同步 硬盘上。





### AOF追加阻塞

当开启AOF持久化时，常用的同步硬盘的策略是everysec，用于平衡性 能和数据安全性。对于这种方式，Redis使用另一条线程每秒执行fsync同步 硬盘。当系统硬盘资源繁忙时，会造成Redis主线程阻塞。

<img src="Redis的持久化.assets/image-20220618180106670.png" alt="image-20220618180106670" style="zoom:50%;" />

通过对AOF阻塞流程可以发现两个问题:

* everysec配置最多可能丢失2秒数据，不是1秒。

* 如果系统fsync缓慢，将会导致Redis主线程阻塞影响效率。：

发生AOF阻塞时，Redis输出如下日志：

> Asynchronous AOF fsync is taking too long (disk is busy). Writing the AOF buffer without waiting for fsync to complete, this may slow down Redis

* 每当发生AOF追加阻塞事件发生时，在info Persistence统计中， aof_delayed_fsync指标会累加，查看这个指标方便定位AOF阻塞问题。
* AOF同步最多允许2秒的延迟，当延迟发生时说明硬盘存在高负载问 题，可以通过监控工具如iotop，定位消耗硬盘IO资源的进程。

优化方式，主要采取减轻磁盘负载的策略，可以参考磁盘优化的相关策略。





## 多实例部署

Redis单线程架构导致无法充分利用CPU多核特性，通常的做法是在一 台机器上部署多个Redis实例。当多个实例开启AOF重写后，彼此之间会产 生对CPU和IO的竞争。本节主要介绍针对这种场景的分析和优化。

上一节介绍了持久化相关的子进程开销。对于单机多Redis部署，如果 同一时刻运行多个子进程，对当前系统影响将非常明显，因此需要采用一种 措施，把子进程工作进行隔离。Redis在info Persistence中为我们提供了监控 子进程运行状况的度量指标，如表所示。

<img src="Redis的持久化.assets/image-20220618180834493.png" alt="image-20220618180834493" style="zoom:50%;" />

我们基于以上指标，可以通过外部程序轮询控制AOF重写操作的执行， 整个过程如图所示。

<img src="Redis的持久化.assets/image-20220618180853121.png" alt="image-20220618180853121" style="zoom:50%;" />

流程说明:

1. 外部程序定时轮询监控机器(machine)上所有Redis实例。
2. 对于开启AOF的实例，查看(aof_current_size- aof_base_size)/aof_base_size确认增长率。
3. 当增长率超过特定阈值(如100%)，执行bgrewriteaof命令手动触发 当前实例的AOF重写。
4. 运行期间循环检查aof_rewrite_in_progress和 aof_current_rewrite_time_sec指标，直到AOF重写结束。
5. 确认实例AOF重写完成后，再检查其他实例并重复2)~4)步操作。 从而保证机器内每个Redis实例AOF重写串行化执行

该过程可能要配置禁止AOF自动重写，由外部调度来决定哪个Redis实例应该进行AOF重写，保证同一时刻，只有一个Redis实例在进行AOF重写，从而避免对计算机资源的竞争。



参考书籍：

《redis开发与运维》













