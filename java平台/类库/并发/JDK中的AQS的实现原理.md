多个线程访问同一个共享变量，可以用

* sychronized
* CAS
* 并发安全的数据结构
* Lock（比如ReetrantLock）

其中，Lock的底层是使用的是AQS（Abstract Quueu State）这种技术。例如，有如下这段代码：

```java
ReentrantLock lock = new ReentrantLock(true);

// 多个线程过来，都尝试
lock.lock();
// Do sth.
lock.unlock();
```

多个线程过来调用lock()的时候，底层就会使用AQS，AQS的原理如下图所示：

![image-20210727112518499](JDK%E4%B8%AD%E7%9A%84AQS%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/image-20210727112518499.png)

1. 线程1和线程2同时并发过来
2. 线程1对state进行CAS操作，并且成功了，然后把加锁线程的这个变量更新为自己。
3. 线程2对state进行CAS操作，由于线程1先操作了，所以线程2失败了，此时线程2会被放入等待队列
4. 线程1执行完后，会把state编程0，加锁线程编程null，然后唤醒等待队列中的第一个线程

**非公平锁**，如果此时来了一个线程3，线程3是可以先于被唤醒的线程3对state进行CAS操作的。

**公平锁**，线程3会在排在等待队列中。















