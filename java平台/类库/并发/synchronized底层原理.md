## Synchronized关键字底层原理

实际上，这里只记录稍微底层一点的原理，并没有很深。

其实synchronized底层的原理，是跟jvm指令和monitor有关系的，你如果用到了synchronized关键字，在底层编译后的jvm指令中，会有monitorenter和monitorexit两个指令

```
monitorenter
// 代码对应的指令
monitorexit
```

那么monitorenter指令执行的时候会干什么呢？

每个对象都有一个关联的**monitor**，比如一个对象实例就有一个monitor，一个类的Class对象也有一个monitor，如果要对这个对象加锁，那么必须获取这个对象关联的monitor的lock锁。

他里面的原理和思路大概是这样的，monitor里面有一个计数器，从0开始的。如果一个线程要获取monitor的锁，就看看他的计数器是不是0，如果是0的话，那么说明没人获取锁，他就可以获取锁了，然后对计数器加1

 这个monitor的锁是支持重入加锁的，什么意思呢，好比下面的代码片段

```java
// 线程1
synchronized(myObject) { -> 类的class对象来走的
		// 一大堆的代码
    synchronized(myObject) {
    	// 一大堆的代码
    }
}
```

加锁，一般来说都是必须对一个对象进行加锁

如果一个线程第一次synchronized那里，获取到了myObject对象的monitor的锁，计数器加1，然后第二次synchronized那里，会再次获取myObject对象的monitor的锁，这个就是重入加锁了，然后计数器会再次加1，变成2

这个时候，其他的线程在第一次synchronized那里，会发现说myObject对象的monitor锁的计数器是大于0的，意味着被别人加锁了，然后此时线程就会进入block阻塞状态，什么都干不了，就是等着获取锁 

接着如果出了synchronized修饰的代码片段的范围，就会有一个monitorexit的指令，在底层。此时获取锁的线程就会对那个对象的monitor的计数器减1，如果有多次重入加锁就会对应多次减1，直到最后，计数器是0 

然后后面block住阻塞的线程，会再次尝试获取锁，但是只有一个线程可以获取到锁