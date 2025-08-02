必须先理解内存模型。volatile是用来解决：

* 可见性
* 有序性

函件情况下也是可以保证原子性的。

```java
private volatile int data = 0;

new Thread(){
	public void run(){
		data++;
	}
}.start();

new Thread(){
	public void run(){
		while(data == 0){
			Thread.sleep(100);
		}
	}
}.start();
```

![image-20210727133611079](./assets/volatile%E5%85%B3%E9%94%AE%E5%AD%97%E7%9A%84%E5%8E%9F%E7%90%86/image-20210727133611079.png)



**可见性的解决。*

上边代码，两个线程，根据Java的内存模型，可能第一个线程已经将data累加了，但是线程2依然在读线程自己的工作内存中的data的数据，所以线程1对主存中的data所做的改变对于线程2来说是不可见的。

加上volatile关键字之后，线程1更新完主存中的data之后，会把其他线程的工作内存中的值进行一个失效操作。线程2，则会重新从主存中进行read和load操作。



**有序性的解决，他是如何避免指令重排的**

happens-before原则

* 程序次序规则
* 锁定规则
* volatile变量规则
* 传递规则
* 线程启动规则
* 线程中断规则
* 线程终结规则
* 对象终结规则

这些规则制定了，在这些特殊情况下，不允许编译器进行指令重排。给共享变量加上了volatile关键字之后，就满足了volatile变量规则，从而避免了指令重排。



### volatile底层原理

























