## Maven的项目继承

### 1. Maven中的项目继承的概念

Maven当中也存在继承的概念，每一个maven项目都继承自`super pom`。引用官网的一句话：

> The `packaging` type required to be `pom` for *parent* and *aggregation* (multi-module) projects.

也就是说，被继承的项目或者聚合项目的packaging类型需要是`pom`类型。如：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>my-parent</artifactId>
  <version>2.0</version>
  <packaging>pom</packaging>
</project>
```



### 2. 能够被继承的元素

- groupId
- version
- description
- url
- inceptionYear
- organization
- licenses
- developers
- contributors
- mailingLists
- scm
- issueManagement
- ciManagement
- properties
- dependencyManagement
- **dependencies**
- repositories
- pluginRepositories
- build
  - plugin executions with matching ids
  - plugin configuration
  - etc.
- reporting
- profiles



### 3. dependencyManagement元素

在父项目中定义的`dependencyManagement`元素，其作用在于：

>  It is used by a POM to help manage dependency information across all of its children.

也就是说该元素用于管理所有的子项目中的依赖信息。

不同于`dependencies`元素，该元素中定义的所有依赖在子项目中并不会被自动引入，需要子项目在`dependency`标签中写其想要引入的依赖，但是需要写版本号了，因为版本号已经在父项目的pom文件中的`dependencyManagement`中定义好了，这也就是dependencyManagement的作用之一——依赖的版本号的统一管理



### 4.parent元素

通过在子项目中的parent标签，能够定义其继承于哪个父项目。

```xml-dtd
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <parent>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>my-parent</artifactId>
    <version>2.0</version>
    <relativePath>../my-parent</relativePath>
  </parent>
 
  <artifactId>my-project</artifactId>
</project>
```

子项目将会继承所有父项目中能够被继承的元素。



## SpringBoot中是如何利用Maven的parent标签的？

以上说了Maven的项目继承，SpringBoot当中就对项目继承这个特性进行了应用。

众所周知，使用SpringBoot框架的一大好处就是起步依赖，通过起步依赖，spring boot帮我们管理了各个依赖的版本，使各个依赖不会出现版本冲突；另外，帮我们打包了各个依赖让我们不用再像之前那样自己导入一大堆的依赖，只要引入起步依赖的坐标就可以进行web开发了。那么问题来了，SpringBoot是如何实现起步依赖的呢？其答案，正是Maven的项目继承（这里只讨论Maven项目）。

在每一个使用SpringBoot的WEB工程当中，在pom文件中通常都会看见如下配置：

```xml
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.2.RELEASE</version>
</parent>
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
</dependencies>
```

可以看到，parent当中描述了，我们的项目继承自`spring-boot-start-parent`，并且引入了`spring-boot-starter-web`这个依赖。

> 当然啊，我们的项目**是一个（is a）**SpringBoot项目，自然就与spring-boot-start-parent构成了继承关系

再看看spring-boot-start-parent项目的pom文件是如何定义的。

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.2.RELEASE</version>
</parent>
<artifactId>spring-boot-starter-parent</artifactId>
<packaging>pom</packaging>
<name>spring-boot-starter-parent</name>
<properties>
  <java.version>1.8</java.version>
  <resource.delimiter>@</resource.delimiter>
  <maven.compiler.source>${java.version}</maven.compiler.source>
  <maven.compiler.target>${java.version}</maven.compiler.target>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>
```

可以看到`spring-boot-starter-parent`的packaging类型为**pom**，并且通过properties元素定义了一些子项目的通用信息。最后，它继承自`spring-boot-dependencies`。那么，就再看看spring-boot-dependencies。

```xml
<properties>
  <spring-boot.version>2.3.2.RELEASE</spring-boot.version>
  ......
<properties>
<dependencyManagement>
	<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>${spring-boot.version}</version>
  </dependency>
  ......
</dependencyManagement>
```

可以看到，在`spring-boot-dependencies`这个项目中，其pom文件中定义了`spring-boot-starter-web`这个依赖以及依赖的版本，并将其放进了`dependencyManagement`元素中，因此在我们的项目中只需要写明依赖谁即可，而不用再写版本号了。

```xml
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
</dependencies>
```

最后，再来看看`spring-boot-starter-web`项目的pom文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <!-- This module was also published with a richer model, Gradle metadata,  -->
  <!-- which should be used instead. Do not delete the following line which  -->
  <!-- is to indicate to Gradle or any Gradle module metadata file consumer  -->
  <!-- that they should prefer consuming it instead. -->
  <!-- do_not_remove: published-with-gradle-metadata -->
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <version>2.3.2.RELEASE</version>
  <name>spring-boot-starter-web</name>
  <description>Starter for building web, including RESTful, applications using Spring MVC. Uses Tomcat as the default embedded container</description>
  <url>https://spring.io/projects/spring-boot</url>
  <organization>
    <name>Pivotal Software, Inc.</name>
    <url>https://spring.io</url>
  </organization>
  <licenses>
    <license>
      <name>Apache License, Version 2.0</name>
      <url>https://www.apache.org/licenses/LICENSE-2.0</url>
    </license>
  </licenses>
  <developers>
    <developer>
      <name>Pivotal</name>
      <email>info@pivotal.io</email>
      <organization>Pivotal Software, Inc.</organization>
      <organizationUrl>https://www.spring.io</organizationUrl>
    </developer>
  </developers>
  <scm>
    <connection>scm:git:git://github.com/spring-projects/spring-boot.git</connection>
    <developerConnection>scm:git:ssh://git@github.com/spring-projects/spring-boot.git</developerConnection>
    <url>https://github.com/spring-projects/spring-boot</url>
  </scm>
  <issueManagement>
    <system>GitHub</system>
    <url>https://github.com/spring-projects/spring-boot/issues</url>
  </issueManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>2.3.2.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-json</artifactId>
      <version>2.3.2.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <version>2.3.2.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.2.8.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.8.RELEASE</version>
      <scope>compile</scope>
    </dependency>
  </dependencies>
</project>

```

在这个项目中，定义了一个WEB项目所需要的最小依赖集合，通过Maven的依赖传递，我们自己的项目也能获取这个最小集合。



## 总结

* SpringBoot利用Maven的项目继承特性，帮助我们管理了一个SpringBoot项目所有的依赖的版本号，使我们从管理依赖版本从而避免jar包版本不兼容的工作中解放出来。
* SpringBoot利用Maven的依赖传递特性，帮助我们封装了一个WEB项目所需要的依赖最小集合，是我们从自己引入依赖的工作中解放出来。