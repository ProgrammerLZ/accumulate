# Spring AMQP

## AMQP Abstractions

Spring AMQP包含两部分：

* spring-amqp
* spring-rabbit

spring-amqp包含了org.springframework.aqmp.core这个包，此包当中定义了表示核心AMQP模型的类。Spring AMQP是想要提供一个通用的抽象层，这个抽象层不依赖任何指定的AMQP供应商。这样的好处是，面向抽象层开发代码会更具可移植性。这个抽象层将会被特定的模块进行实现，比如：spring-rabbit。

这个抽象层包含了一些类，例如：

* Message(class)
* Exchange(interface)
* Queue(class)
* Binding(class)



## Connection and Resources Management

尽管我们在上一节中描述的AMQP模型是通用的，并且适用于所有实现，但当我们进入资源管理时，**细节是特定于代理实现的**。因此，在本节中，我们将关注仅存在于“spring-rabbit”模块中的代码，因为RabbitMQ暂时是唯一受支持的实现。

`ConnectionFactory`接口是管理连接的核心组件，他的主要职责是提供`org.springframework.amqp.rabbit.connection.Connection`实例，该Connection封装了`com.rabbitmq.client.Connection`。



