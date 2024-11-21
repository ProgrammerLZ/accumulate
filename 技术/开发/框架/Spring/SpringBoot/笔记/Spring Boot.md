## Spring Boot 基础

### SpringBoot工程结构解析

<img src="Spring%20Boot/image-20210106185025926.png" alt="image-20210106185025926" style="zoom:50%;" />

1. mvnwrapper主要用于maven的版本切换，不是maven本身的东西，springboot工程会自动为我们将mvnwrapper安装好。
2. target mvn在构建的时候生成的文件
3. 01primary.iml  idea的工程配置文件
4. Application类
   1. @SpringBootApplication 组合注解
   2. Application类本身就是一个配置类



### Pom文件解析

**打包的时候会打两个包**

<img src="Spring%20Boot/image-20210106191049045.png" alt="image-20210106191049045" style="zoom:50%;" />

是因为maven的插件的repackage。



### 基于war的工程

**方法一**

创建工程的时候选择使用war

<img src="Spring%20Boot/image-20210106194006884.png" alt="image-20210106194006884" style="zoom:50%;" />

**方法二**

在基于jar的工程上进行修改

<img src="Spring%20Boot/image-20210106195249215.png" alt="image-20210106195249215" style="zoom:50%;" />



### yml文件解析

<img src="Spring%20Boot/image-20210106200013663.png" alt="image-20210106200013663" style="zoom:50%;" />



### Actuator

#### 默认的监控终端

<img src="Spring%20Boot/image-20210106202228976.png" alt="image-20210106202228976" style="zoom:50%;" />



#### 配置监控终端

<img src="Spring%20Boot/image-20210106203606987.png" alt="image-20210106203606987" style="zoom:50%;" />



## 工程应用



### 自定义异常页面

<img src="Spring%20Boot/image-20210106205651314.png" alt="image-20210106205651314" style="zoom:50%;" />



### 一个工程启动多个进程

![image-20210106210501911](Spring%20Boot/image-20210106210501911.png)



### 多配置文件多环境选择实现

```shell
java -jar xxx.jar --spring.profiles.active=dev
```

或

<img src="Spring%20Boot/image-20210107203837135.png" alt="image-20210107203837135" style="zoom:50%;" />

### 单配置文件式多环境实现

<img src="Spring%20Boot/image-20210107203554582.png" alt="image-20210107203554582" style="zoom:50%;" />



### 读取自定义配置

**方法一**

@Value("{server.port}") 取配置文件中定义的端口号。

**方法二**

![image-20210107204727728](Spring%20Boot/image-20210107204727728.png)

由上边的SpringBoot官方文档中的一段话，自定义的配置要放在properties文件中。

<img src="Spring%20Boot/image-20210107210029873.png" alt="image-20210107210029873" style="zoom:50%;" />

<img src="Spring%20Boot/image-20210107210147471.png" alt="image-20210107210147471" style="zoom:50%;" />

<img src="Spring%20Boot/image-20210107210913132.png" alt="image-20210107210913132" style="zoom:50%;" />



**方法二进阶**

**1**

<img src="Spring%20Boot/image-20210107210935893.png" alt="image-20210107210935893" style="zoom:50%;" />

<img src="Spring%20Boot/image-20210107211013620.png" alt="image-20210107211013620" style="zoom:50%;" />

**2**

<img src="Spring%20Boot/image-20210107211453127.png" alt="image-20210107211453127" style="zoom:50%;" />

<img src="Spring%20Boot/image-20210107211526596.png" alt="image-20210107211526596" style="zoom:50%;" />

<img src="Spring%20Boot/image-20210107211614633.png" alt="image-20210107211614633" style="zoom:50%;" />

### Spring Boot中使用Redis

Redis的并发问题：

* 缓存击穿
* 缓存雪崩



### Dubbo 与Spring Boot整合

1. 由`@EnableDubboConfiguration`想到SpringBoot是如何进行自动配置的？
2. 







