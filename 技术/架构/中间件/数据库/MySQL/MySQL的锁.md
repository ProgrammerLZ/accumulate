## 锁类型

### 全局锁

用于给整个实例加锁，加锁的语句为：

```mysql
flush tables with read lock;
```

通常用于Mysql的备份，来保证备份数据的一致性，但是加锁粒度比较大，对业务的影响比较大，因此一般不用该锁，mysqldump提供了--single-transaction参数，也能够达到一致性的目的。 

```shell
mysqldump --single-transaction -uroot -p db > /backup
```



### 表级锁

**表锁**

* 表共享读锁（READ）
  * 本事务可读不可写，其他事务可读不可写
* 表独占写锁（WRITE）
  * 本事务可读写，其他事务不可读不可写

**元数据锁**

* 共享读锁（SHARED_READ）
* 共享写锁（SHARED_WRITE）
* 排他锁（EXCLUSIVE）

下边这张图表达了不同的SQL语句，会给表加的元数据锁以及锁之间的兼容情况

![image-20231126144946056](assets/MySQL%E7%9A%84%E9%94%81/image-20231126144946056.png)

SHARED_READ 和 SHARED_WRITE是兼容的，但是与EXCLUSIVE是互斥的。

**意向锁**

* 意向共享锁（IS）

  * 与表共享读锁兼容
  * 与表独占写锁互斥

* 意向排他锁（IX）

  * 与表共享读锁互斥

  * 与表独占写锁互斥

自动加锁。select ... in share mode加IS锁，update/insert/delete加IX锁。

### 行级锁

* 行锁（S，REC_NOT_GAP）
* 间隙锁（GAP）
* next-key锁（S）



## 背景

### 并发事务

并发事务的问题：

* 更新丢失

* 脏读（dirty reads）

  A事务读取了B事务（**未提交**）正在修改的一条记录，并且A事务根据这条记录的当前状态做接下来的处理

* 不可重复读

  一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现其读出的数据已经发生了改变或某些记录已经被删除了

* 幻读

  一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据



### 事务隔离级别

![image-20231102134921497](assets/MySQL%E7%9A%84%E9%94%81/image-20231102134921497.png)



## 获取InnoDB行锁争用情况

```shell
mysql> show status like 'innodb_row_lock%'
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 1     |
| Innodb_row_lock_time_avg      | 1     |
| Innodb_row_lock_time_max      | 1     |
| Innodb_row_lock_waits         | 1     |
+-------------------------------+-------+
```

Innodb_row_lock_current_waits，Innodb_row_lock_time_avg这两个值比较高，可以通过

* 查询information_schema 数据库中相关的表来查看锁情况

  ```mysql
  select * from innodb_locks;
  select * from innodb_lock_waits;
  ```

* 或者通过设置InnoDB Monitors来进一步观察发生锁冲突的表、数据行等，并分析锁争用的原因

  ```mysql
  CREATE TABLE innodb_monitor(a INT) ENGINE=INNODB;
  # 要记得删除监控表以关闭监视器
  DROP TABLE innodb_monitor;
  ```

  

## 行锁模式以及加锁方法

InnoDB实现了以下两种类型的行锁。

共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。	

排他锁（X）：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享锁和排他锁。



InnoDB 还有两种内部使用的意向锁（Intention Locks），这两种意向锁都是表锁。

意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。

意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁。



上述锁的兼容情况：

![image-20231102143846030](assets/MySQL%E7%9A%84%E9%94%81/image-20231102143846030.png)



对于加锁时机，意向锁是InnoDB自动加的，不需用户干预；对于UPDATE、DELETE和INSERT语句， InnoDB会自动给涉及数据集加排他锁（X）；对于普通SELECT语句，InnoDB不会加任何锁。如果在select的时候也想加锁，可以通过以下方式：

```shell
# 加共享锁
SELECT * FROM table_name WHERE ...LOCK IN SHARE MODE
# 加排他锁
SELECT * FROM table_name WHERE ... FOR UPDATE
```



## InnoDB行锁实现方式

InnoDB行锁是通过**给索引上的索引项加锁**来实现的，如果没有索引，InnoDB将通过隐藏的聚簇索引来对记录加锁。

InnoDB行锁分为3种情形。

* Record lock：对索引项加锁，防止其他事务对此行进行update/insert/delete操作。在RC和RR隔离级别下都支持。

* Gap lock：对索引项之间的“间隙”、第一条记录前的“间隙”或最后一条记录后的“间隙”加锁。在RR隔离级别下支持。

* Next-key lock：前两种的组合，对记录及其前面的间隙加锁。在RR隔离级别下支持。

在使用InnoDB的行锁的时候，有以下一些需要注意的地方：

* **在不通过索引条件查询时，InnoDB会锁定表中的所有记录**
* 由于 MySQL 的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现锁冲突的。应用设计的时候要注意这一点。
* 当表有多个索引的时候，不同的事务可以使用不同的索引锁定不同的行，不论是使用主键索引、唯一索引或普通索引，InnoDB都会使用行锁来对数据加锁
* 即便在条件中使用了索引字段，但是否使用索引来检索数据是由MySQL通过判断不同执行计划的代价来决定的，如果MySQL认为全表扫描效率更高，比如对一些很小的表，它就不会使用索引，这种情况下InnoDB也会对所有记录加锁。因此，在分析锁冲突时，别忘了检查SQL的执行计划，以确认是否真正使用了索引。关于MySQL在什么情况下不使用索引的详细讨论



## Next-key锁

当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB 会给符合条件的已有数据记录的索引项加锁；**对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP）”**，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的Next-Key锁。

InnoDB使用Next-Key锁的目的：

* 一方面是为了防止幻读，以满足相关隔离级别的要求
* 为了满足其恢复和复制的需要

> InnoDB除了通过范围条件加锁时使用Next-Key锁外，如果**使用相等条件请求给一个不存在的记录加锁**，InnoDB也会使用Next-Key锁

MySQL使用binlog实现数据库的恢复和主从复制，MySQL 5.6的binlog有3种日志模式:

* 基于语句的日志格式SBL
* 基于行的日志格式RBL
* 混合格式

MySLQ有4种复制模式：

* 基于SQL语句的复制SBR
* 基于行数据的复制RBR
* 混合复制模式
* 使用全局事务ID（GTIDs）的复制

首先，对基于语句日志格式（SBL）的恢复和复制而言，MySQL的binlog是**按照事务提交的先后顺序记录**的。

考虑以下情况：

1. 假设一个隔离级别为可重复读的事务A，要执行一个范围更新T表的操作，根据更新条件，可更新10条数据。
2. 同样的，假设一个隔离级别为可重复读的事务B，也更新T表，重要的是其更新了事务A要更新的那10条数据，且其更新的列为A的查询条件，最后其先于A事务提交。
3. 事务A，最终也提交
4. binlog中先记录了B事务的更新语句，后记录了A事务的更新语句
5. 此时，数据库崩溃，需要对数据库进行恢复和复制
6. 使用binlog进行 恢复的时候发现，事务A最终能够更新的数据不足10条，且发生了主从数据库不一致的问题

因此，MySQL会给这10条数据加锁，以满足让A事务先来获取到这10条数据的使用权，B事务等待，从而保证binlog中的事务顺序，最终保证数据库的恢复和复制能够成功进行。所以，可以看到，即使在可重复读这个隔离级别下，虽然是允许产生幻读的，但由于MySQL为了解决上述问题从而给这10条数据加了锁，所以对于这10条数据来说永远都不可能产生幻读。





























