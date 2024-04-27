## [Spring AOP](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop)

We have already seen how [Spring Dependency Injection](https://www.digitalocean.com/community/tutorials/spring-dependency-injection) works, today we will look into the core concepts of Aspect-Oriented Programming and how we can implement it using Spring Framework.

### [Spring AOP Overview](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-overview)

Most of the enterprise applications have some common crosscutting concerns that are applicable to different types of Objects and modules. Some of the common crosscutting concerns are logging, transaction management, data validation, etc. In Object Oriented Programming, modularity of application is achieved by Classes whereas in Aspect Oriented Programming application modularity is achieved by Aspects and they are configured to cut across different classes. Spring AOP takes out the direct dependency of crosscutting tasks from classes that we can’t achieve through normal object oriented programming model. For example, we can have a separate class for logging but again the functional classes will have to call these methods to achieve logging across the application.

### [Aspect Oriented Programming Core Concepts](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#aspect-oriented-programming-core-concepts)

Before we dive into the implementation of Spring AOP implementation, we should understand the core concepts of AOP.

1. **Aspect**: An aspect is a class that implements enterprise application concerns that cut across multiple classes, such as transaction management. Aspects can be a normal class configured through Spring XML configuration or we can use Spring AspectJ integration to define a class as Aspect using `@Aspect` annotation.
2. **Join Point**: A join point is a specific point in the application such as method execution, exception handling, changing object variable values, etc. In Spring AOP a join point is always the execution of a method.
3. **Advice**: Advices are actions taken for a particular join point. In terms of programming, they are methods that get executed when a certain join point with matching pointcut is reached in the application. You can think of Advices as [Struts2 interceptors](https://www.digitalocean.com/community/tutorials/struts-2-interceptor-example) or [Servlet Filters](https://www.digitalocean.com/community/tutorials/java-servlet-filter-example-tutorial).
4. **Pointcut**: Pointcut is expressions that are matched with join points to determine whether advice needs to be executed or not. Pointcut uses different kinds of expressions that are matched with the join points and Spring framework uses the AspectJ pointcut expression language.
5. **Target Object**: They are the object on which advices are applied. Spring AOP is implemented using runtime proxies so this object is always a proxied object. What is means is that a subclass is created at runtime where the target method is overridden and advice are included based on their configuration.
6. **AOP proxy**: Spring AOP implementation uses JDK dynamic proxy to create the Proxy classes with target classes and advice invocations, these are called AOP proxy classes. We can also use CGLIB proxy by adding it as the dependency in the Spring AOP project.
7. **Weaving**: It is the process of linking aspects with other objects to create the advised proxy objects. This can be done at compile time, load time or at runtime. Spring AOP performs weaving at the runtime.

### [AOP Advice Types](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#aop-advice-types)

Based on the execution strategy of advice, they are of the following types.

1. **Before Advice**: These advices runs before the execution of join point methods. We can use `@Before` annotation to mark an advice type as Before advice.
2. **After (finally) Advice**: An advice that gets executed after the join point method finishes executing, whether normally or by throwing an exception. We can create after advice using `@After` annotation.
3. **After Returning Advice**: Sometimes we want advice methods to execute only if the join point method executes normally. We can use `@AfterReturning` annotation to mark a method as after returning advice.
4. **After Throwing Advice**: This advice gets executed only when join point method throws exception, we can use it to rollback the transaction declaratively. We use `@AfterThrowing` annotation for this type of advice.
5. **Around Advice**: This is the most important and powerful advice. This advice surrounds the join point method and we can also choose whether to execute the join point method or not. We can write advice code that gets executed before and after the execution of the join point method. It is the responsibility of around advice to invoke the join point method and return values if the method is returning something. We use `@Around` annotation to create around advice methods.

The points mentioned above may sound confusing but when we will look at the implementation of Spring AOP, things will be more clear. Let’s start creating a simple Spring project with AOP implementations. Spring provides support for using AspectJ annotations to create aspects and we will be using that for simplicity. All the above AOP annotations are defined in `org.aspectj.lang.annotation` package. **Spring Tool Suite** provides useful information about the aspects, so I would suggest you use it. If you are not familiar with STS, I would recommend you to have a look at [Spring MVC Tutorial](https://www.digitalocean.com/community/tutorials/spring-mvc-tutorial) where I have explained how to use it.

## [Spring AOP Example](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-example)

Create a new Simple Spring Maven project so that all the Spring Core libraries are included in the pom.xml files and we don’t need to include them explicitly. Our final project will look like the below image, we will look into the Spring core components and Aspect implementations in detail.[![Spring AOP Example Tutorial, Spring Aspect Example](assets/Spring%20AOP/Spring-AOP-Example-Project.png)](https://journaldev.nyc3.cdn.digitaloceanspaces.com/2014/03/Spring-AOP-Example-Project.png)

### [Spring AOP AspectJ Dependencies](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-aspectj-dependencies)

Spring framework provides AOP support by default but since we are using AspectJ annotations for configuring aspects and advice, we would need to include them in the pom.xml file.

```xml
<project xmlns="https://maven.apache.org/POM/4.0.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="https://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.springframework.samples</groupId>
	<artifactId>SpringAOPExample</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>

		<!-- Generic properties -->
		<java.version>1.6</java.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

		<!-- Spring -->
		<spring-framework.version>4.0.2.RELEASE</spring-framework.version>

		<!-- Logging -->
		<logback.version>1.0.13</logback.version>
		<slf4j.version>1.7.5</slf4j.version>

		<!-- Test -->
		<junit.version>4.11</junit.version>

		<!-- AspectJ -->
		<aspectj.version>1.7.4</aspectj.version>

	</properties>

	<dependencies>
		<!-- Spring and Transactions -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${spring-framework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring-framework.version}</version>
		</dependency>

		<!-- Logging with SLF4J & LogBack -->
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
			<scope>compile</scope>
		</dependency>
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-classic</artifactId>
			<version>${logback.version}</version>
			<scope>runtime</scope>
		</dependency>

		<!-- AspectJ dependencies -->
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjrt</artifactId>
			<version>${aspectj.version}</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjtools</artifactId>
			<version>${aspectj.version}</version>
		</dependency>
	</dependencies>
</project>
```

Notice that I have added `aspectjrt` and `aspectjtools` dependencies (version 1.7.4) in the project. Also I have updated the Spring framework version to be the latest one as of date i.e 4.0.2.RELEASE.

### [Model Class](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#model-class)

Let’s create a simple java bean that we will use for our example with some additional methods. Employee.java code:

```java
package com.journaldev.spring.model;

import com.journaldev.spring.aspect.Loggable;

public class Employee {

	private String name;
	
	public String getName() {
		return name;
	}

	@Loggable
	public void setName(String nm) {
		this.name=nm;
	}
	
	public void throwException(){
		throw new RuntimeException("Dummy Exception");
	}	
}
```

Did you noticed that *setName()* method is annotated with `Loggable` annotation. It is a [custom java annotation](https://www.digitalocean.com/community/tutorials/java-annotations) defined by us in the project. We will look into it’s usage later on.

### [Service Class](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#service-class)

Let’s create a service class to work with the Employee bean. EmployeeService.java code:

```java
package com.journaldev.spring.service;

import com.journaldev.spring.model.Employee;

public class EmployeeService {

	private Employee employee;
	
	public Employee getEmployee(){
		return this.employee;
	}
	
	public void setEmployee(Employee e){
		this.employee=e;
	}
}
```

I could have used Spring annotations to configure it as a Spring Component, but we will use XML based configuration in this project. EmployeeService class is very standard and just provides us an access point for Employee beans.

### [Spring Bean Configuration with AOP](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-bean-configuration-with-aop)

If you are using STS, you have the option to create “Spring Bean Configuration File” and chose AOP schema namespace but if you are using some other IDE, you can simply add it in the spring bean configuration file. My project bean configuration file looks like below. spring.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="https://www.springframework.org/schema/beans"
	xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="https://www.springframework.org/schema/aop"
	xsi:schemaLocation="https://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		https://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

<!-- Enable AspectJ style of Spring AOP -->
<aop:aspectj-autoproxy />

<!-- Configure Employee Bean and initialize it -->
<bean name="employee" class="com.journaldev.spring.model.Employee">
	<property name="name" value="Dummy Name"></property>
</bean>

<!-- Configure EmployeeService bean -->
<bean name="employeeService" class="com.journaldev.spring.service.EmployeeService">
	<property name="employee" ref="employee"></property>
</bean>

<!-- Configure Aspect Beans, without this Aspects advices wont execute -->
<bean name="employeeAspect" class="com.journaldev.spring.aspect.EmployeeAspect" />
<bean name="employeeAspectPointcut" class="com.journaldev.spring.aspect.EmployeeAspectPointcut" />
<bean name="employeeAspectJoinPoint" class="com.journaldev.spring.aspect.EmployeeAspectJoinPoint" />
<bean name="employeeAfterAspect" class="com.journaldev.spring.aspect.EmployeeAfterAspect" />
<bean name="employeeAroundAspect" class="com.journaldev.spring.aspect.EmployeeAroundAspect" />
<bean name="employeeAnnotationAspect" class="com.journaldev.spring.aspect.EmployeeAnnotationAspect" />

</beans>
```

For using Spring AOP in Spring beans, we need to do the following:

1. Declare AOP namespace like xmlns:aop=“https://www.springframework.org/schema/aop”
2. Add aop:aspectj-autoproxy element to enable Spring AspectJ support with auto proxy at runtime
3. Configure Aspect classes as other Spring beans

You can see that I have a lot of aspects defined in the spring bean configuration file, it’s time to look into them one by one.

### [Spring AOP Before Aspect Example](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-before-aspect-example)

EmployeeAspect.java code:

```java
package com.journaldev.spring.aspect;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class EmployeeAspect {

	@Before("execution(public String getName())")
	public void getNameAdvice(){
		System.out.println("Executing Advice on getName()");
	}
	
	@Before("execution(* com.journaldev.spring.service.*.get*())")
	public void getAllAdvice(){
		System.out.println("Service method getter called");
	}
}
```

Important points in above aspect class are:

- **Aspect** classes are required to have `@Aspect` annotation.
- [@Before](https://www.digitalocean.com/community/users/before) annotation is used to create Before advice
- The string parameter passed in the `@Before` annotation is the Pointcut expression
- *getNameAdvice()* advice will execute for any Spring Bean method with signature `public String getName()`. This is a very important point to remember, if we will create Employee bean using new operator the advices will not be applied. Only when we will use ApplicationContext to get the bean, advices will be applied.
- We can use asterisk (*) as wild card in Pointcut expressions, *getAllAdvice()* will be applied for all the classes in `com.journaldev.spring.service` package whose name starts with `get` and doesn’t take any arguments.

We will look at the advice in action in a test class after we have looked into all the different types of advices.

### [Spring AOP Pointcut Methods and Reuse](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-pointcut-methods-and-reuse)

Sometimes we have to use same Pointcut expression at multiple places, we can create an empty method with `@Pointcut` annotation and then use it as an expression in the advices. EmployeeAspectPointcut.java code:

```java
package com.journaldev.spring.aspect;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class EmployeeAspectPointcut {

	@Before("getNamePointcut()")
	public void loggingAdvice(){
		System.out.println("Executing loggingAdvice on getName()");
	}
	
	@Before("getNamePointcut()")
	public void secondAdvice(){
		System.out.println("Executing secondAdvice on getName()");
	}
	
	@Pointcut("execution(public String getName())")
	public void getNamePointcut(){}
	
	@Before("allMethodsPointcut()")
	public void allServiceMethodsAdvice(){
		System.out.println("Before executing service method");
	}
	
	//Pointcut to execute on all the methods of classes in a package
	@Pointcut("within(com.journaldev.spring.service.*)")
	public void allMethodsPointcut(){}
	
}
```

Above example is very clear, rather than expression we are using method name in the advice annotation argument.

### [Spring AOP JoinPoint and Advice Arguments](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-joinpoint-and-advice-arguments)

We can use JoinPoint as a parameter in the advice methods and using it get the method signature or the target object. We can use `args()` expression in the pointcut to be applied to any method that matches the argument pattern. If we use this, then we need to use the same name in the advice method from where the argument type is determined. We can use [Generic objects](https://www.digitalocean.com/community/tutorials/java-generics-example-method-class-interface) also in the advice arguments. EmployeeAspectJoinPoint.java code:

```java
package com.journaldev.spring.aspect;

import java.util.Arrays;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class EmployeeAspectJoinPoint {
	
	@Before("execution(public void com.journaldev.spring.model..set*(*))")
	public void loggingAdvice(JoinPoint joinPoint){
		System.out.println("Before running loggingAdvice on method="+joinPoint.toString());
		
		System.out.println("Agruments Passed=" + Arrays.toString(joinPoint.getArgs()));

	}
	
	//Advice arguments, will be applied to bean methods with single String argument
	@Before("args(name)")
	public void logStringArguments(String name){
		System.out.println("String argument passed="+name);
	}
}
```

### [Spring AOP After Advice Example](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-after-advice-example)

Let’s look at a simple aspect class with an example of After, After Throwing and After Returning advice. EmployeeAfterAspect.java code:

```java
package com.journaldev.spring.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class EmployeeAfterAspect {

	@After("args(name)")
	public void logStringArguments(String name){
		System.out.println("Running After Advice. String argument passed="+name);
	}
	
	@AfterThrowing("within(com.journaldev.spring.model.Employee)")
	public void logExceptions(JoinPoint joinPoint){
		System.out.println("Exception thrown in Employee Method="+joinPoint.toString());
	}
	
	@AfterReturning(pointcut="execution(* getName())", returning="returnString")
	public void getNameReturningAdvice(String returnString){
		System.out.println("getNameReturningAdvice executed. Returned String="+returnString);
	}
	
}
```

We can use `within` in pointcut expression to apply the advice to all the methods in the class. We can use [@AfterReturning](https://www.digitalocean.com/community/users/afterreturning) advice to get the object returned by the advised method. We have *throwException()* method in the Employee bean to showcase the use of After Throwing advice.

### [Spring AOP Around Aspect Example](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-around-aspect-example)

As explained earlier, we can use Around aspect to cut the method execution before and after. We can use it to control whether the advised method will execute or not. We can also inspect the returned value and change it. This is the most powerful advice and needs to be applied properly. EmployeeAroundAspect.java code:

```java
package com.journaldev.spring.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class EmployeeAroundAspect {

	@Around("execution(* com.journaldev.spring.model.Employee.getName())")
	public Object employeeAroundAdvice(ProceedingJoinPoint proceedingJoinPoint){
		System.out.println("Before invoking getName() method");
		Object value = null;
		try {
			value = proceedingJoinPoint.proceed();
		} catch (Throwable e) {
			e.printStackTrace();
		}
		System.out.println("After invoking getName() method. Return value="+value);
		return value;
	}
}
```

Around advice are always required to have ProceedingJoinPoint as an argument and we should use it’s proceed() method to invoke the target object advised method. If advised method is returning something, it’s advice responsibility to return it to the caller program. For void methods, advice method can return null. Since around advice cut around the advised method, we can control the input and output of the method as well as it’s execution behavior.

### [Spring Advice with Custom Annotation Pointcut](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-advice-with-custom-annotation-pointcut)

If you look at all the above advice pointcut expressions, there are chances that they get applied to some other beans where it’s not intended. For example, someone can define a new spring bean with getName() method and the advice will start getting applied to that even though it was not intended. That’s why we should keep the scope of pointcut expression as narrow as possible. An alternative approach is to create a custom annotation and annotate the methods where we want the advice to be applied. This is the purpose of having Employee *setName()* method annotated with [@Loggable](https://www.digitalocean.com/community/users/loggable) annotation. Spring Framework [@Transactional](https://www.digitalocean.com/community/users/transactional) annotation is a great example of this approach for [Spring Transaction Management](https://www.digitalocean.com/community/tutorials/spring-transaction-management-jdbc-example). Loggable.java code:

```java
package com.journaldev.spring.aspect;

public @interface Loggable {

}
```

EmployeeAnnotationAspect.java code:

```java
package com.journaldev.spring.aspect;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class EmployeeAnnotationAspect {

	@Before("@annotation(com.journaldev.spring.aspect.Loggable)")
	public void myAdvice(){
		System.out.println("Executing myAdvice!!");
	}
}
```

The myAdvice() method will advice only setName() method. This is a very safe approach and whenever we want to apply the advice on any method, all we need is to annotate it with Loggable annotation.

### [Spring AOP XML Configuration](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-xml-configuration)

I always prefer annotation but we also have the option to configure aspects in the spring configuration file. For example, let’s say we have a class as below. EmployeeXMLConfigAspect.java code:

```java
package com.journaldev.spring.aspect;

import org.aspectj.lang.ProceedingJoinPoint;

public class EmployeeXMLConfigAspect {

	public Object employeeAroundAdvice(ProceedingJoinPoint proceedingJoinPoint){
		System.out.println("EmployeeXMLConfigAspect:: Before invoking getName() method");
		Object value = null;
		try {
			value = proceedingJoinPoint.proceed();
		} catch (Throwable e) {
			e.printStackTrace();
		}
		System.out.println("EmployeeXMLConfigAspect:: After invoking getName() method. Return value="+value);
		return value;
	}
}
```

We can configure it by including the following configuration in the Spring Bean config file.

```java
<bean name="employeeXMLConfigAspect" class="com.journaldev.spring.aspect.EmployeeXMLConfigAspect" />

<!-- Spring AOP XML Configuration -->
<aop:config>
	<aop:aspect ref="employeeXMLConfigAspect" id="employeeXMLConfigAspectID" order="1">
		<aop:pointcut expression="execution(* com.journaldev.spring.model.Employee.getName())" id="getNamePointcut"/>
		<aop:around method="employeeAroundAdvice" pointcut-ref="getNamePointcut" arg-names="proceedingJoinPoint"/>
	</aop:aspect>
</aop:config>
```

AOP xml config elements purpose is clear from their name, so I won’t go into much detail about it.

### [Spring AOP Example](https://www.digitalocean.com/community/tutorials/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations#spring-aop-example)

Let’s have a simple Spring program and see how all these aspects cut through the bean methods. SpringMain.java code:

```java
package com.journaldev.spring.main;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.journaldev.spring.service.EmployeeService;

public class SpringMain {

	public static void main(String[] args) {
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("spring.xml");
		EmployeeService employeeService = ctx.getBean("employeeService", EmployeeService.class);
		
		System.out.println(employeeService.getEmployee().getName());
		
		employeeService.getEmployee().setName("Pankaj");
		
		employeeService.getEmployee().throwException();
		
		ctx.close();
	}
}
```

Now when we execute the above program, we get the following output.

```shell
Mar 20, 2014 8:50:09 PM org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
INFO: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@4b9af9a9: startup date [Thu Mar 20 20:50:09 PDT 2014]; root of context hierarchy
Mar 20, 2014 8:50:09 PM org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
INFO: Loading XML bean definitions from class path resource [spring.xml]
Service method getter called
Before executing service method
EmployeeXMLConfigAspect:: Before invoking getName() method
Executing Advice on getName()
Executing loggingAdvice on getName()
Executing secondAdvice on getName()
Before invoking getName() method
After invoking getName() method. Return value=Dummy Name
getNameReturningAdvice executed. Returned String=Dummy Name
EmployeeXMLConfigAspect:: After invoking getName() method. Return value=Dummy Name
Dummy Name
Service method getter called
Before executing service method
String argument passed=Pankaj
Before running loggingAdvice on method=execution(void com.journaldev.spring.model.Employee.setName(String))
Agruments Passed=[Pankaj]
Executing myAdvice!!
Running After Advice. String argument passed=Pankaj
Service method getter called
Before executing service method
Exception thrown in Employee Method=execution(void com.journaldev.spring.model.Employee.throwException())
Exception in thread "main" java.lang.RuntimeException: Dummy Exception
	at com.journaldev.spring.model.Employee.throwException(Employee.java:19)
	at com.journaldev.spring.model.Employee$$FastClassBySpringCGLIB$$da2dc051.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:711)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
	at org.springframework.aop.aspectj.AspectJAfterThrowingAdvice.invoke(AspectJAfterThrowingAdvice.java:58)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:644)
	at com.journaldev.spring.model.Employee$$EnhancerBySpringCGLIB$$3f881964.throwException(<generated>)
	at com.journaldev.spring.main.SpringMain.main(SpringMain.java:17)
```

You can see that advices are getting executed one by one based on their pointcut configurations. You should configure them one by one to avoid confusion. That’s all for **Spring AOP Example Tutorial**, I hope you learned the basics of AOP with Spring and can learn more from examples. Download the sample project from below link and play around with it.