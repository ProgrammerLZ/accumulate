

##  CAS（Compare and set）的基本原理

```java
public class MyObject{
	AtomicInteger i = new AtomicInteger(0);//底层是基于ACS的
	
	pubic void increment(){
		i.incrementAndGet();
	}
}
```

synchronized关键字，加锁的粒度比较大，会导致程序的整体效率变低，因此，为了减小加锁的粒度，JDK的并发包中的一些类，采用了CAS这个技术，CAS能够在硬件级别保证一个操作一定是原子的。比如上述代码中的`incrementAndGet`这个方法，其内部实现就有一个操作是利用了CAS这项技术，原理如下：

![CAS](CAS%E7%9A%84%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86/8959800_1578391025.png)

对于每一个线程，incrementAndGet会先获取其内部的值，然后进行累加操作，最后一个步骤是一个CAS（Compare and set）操作，会先跟原来的值进行比较，如果原来的值 = 累加后的值- 1，那么就是可以正常往回set的，否则则会重新再来一此上述的操作。