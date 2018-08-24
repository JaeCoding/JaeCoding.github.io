---
title: Spring
date: 2018-08-19 09:38:37
tags: [操作系统,IO]
categories: 原理

---

本文全片以问题为导向，阐述Spring的意义

# Spring实战

# Spring的目的

简化Java开发
问题：很多框架通过强迫应用**继承它们的类或实现它们的接口**从
而导致应用与框架绑死。（侵入式）

Spring构建的应用中，它的类通常没有任何痕迹表明你使用了Spring。最坏的场景是，一个类或许会使用**Spring注解**，但它依旧是POJO。

### 1.1.2 依赖注入
类相互之间进行**协作**来完成特定的业务逻辑。就会产生依赖

**问题：**传统的做法，每个对象负责管理与自己相互协作的对象（即它所依赖的对象）的引用，这将会导致**高度耦合**和**难以测试**的代码。
**解决：**通过DI，对象的依赖关系将由系统中负责协调各对象的第三方组件在创建对象的时候进行设定。对象无需自行创建或管理它们的依赖关系。
**总结：**任务对象不是在骑士对象中创建，并引用，而是通过注入骑士对象来引用，创建与使用分开。

**装配：**如何将任务注入骑士？
1.使用**应用上下文（Application Context）xml**，配置id，class与constructor-arg。被声明为Spring中的bean。就BraveKnightbean来讲，它在构造时传入了对SlayDragonQuest bean的引用，将其作为构造器参数。

**使用：** ClassPathXmlApplicationContext加载knights.xml，并获得
Knight对象的引用

2.Java @Configuration**注解配置**

### 1.1.3 应用切面AOP
**作用：** AOP允许你把遍布应用各处的**功能分离**出来形成**可重用的组件**。

服务：如日志、事务管理和安全这样的系统服务

**问题：**
 - 1.实现系统关注点功能的代码将会**重复出现**在多个组件中。若要修改这些关注点，需要修改所有模块中的实现
 - 2.组件会因为那些与自身核心业务无关的**代码而变得混乱**。

**解决：** AOP以声明的方式将它们应用到它们需要影响的组件中去，使用各种**功能层**去包裹**核心业务层**。
 
**实现:**在spring配置中，声明**切面bean**，并且配置aop：config引入切面bean。主要有定义切点，声明前置通知，声明后置通知。

### 1.1.4 使用 **模板** 消除样板式代码（虽然还是不够简洁）

**问题：**在使用JDBC的时候，每个SQL都需要写一大堆样板式代码，然后一大堆清理。
**解决：** Spring的**JdbcTemplate**重写的getEmployeeById()方法仅仅关注于获取员工数据的核心逻辑，而不需要迎合JDBC API的需求。

## Spring Bean容器

Spring容器并不是只有一个。Spring自带了多个容器实现，可以归为
两种不同的类型。

 - **bean工厂**（由org.springframework. beans. factory.BeanFactory接口定义）是最简单的容器，提供基本的DI 支持。
 - **应用上下文**（常用，更受欢迎）（由org.springframework.context.ApplicationContext接口定义）基于BeanFactory构建，并提供应用框架级别的服务，例如 从**属性文件解析文本信息**以及发布应用事件给感兴趣的事件监听者。
 
**多种应用上下文**
 - AnnotationConfigApplicationContext：从一个或多个 基于**Java的配置类**中加载**Spring应用**上下文。 
 - AnnotationConfigWebApplicationContext：从一个或 多个基于**Java的配置类**中加载**SpringWeb应用**上下文。 
 - ClassPathXmlApplicationContext：从类路径下的一个或多个**XML配置文件**中加载上下文定义，把应用上下文的定义文件作为类资源。
 - FileSystemXmlapplicationcontext：从**文件系统**下的一 个或多个**XML配置**文件中加载上下文定义。
 - XmlWebApplicationContext：从**Web应用下**的一个或多个 **XML配置**文件中加载上下文定义。
 
### 1.2.2 bean的生命周期
一般：new -> 使用 -> GC
Spring：
![此处输入图片的描述][1]
  [1]: http://paxblmm0h.bkt.clouddn.com/bean%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png

实例化->填充属性->传id，传beanFactory，传上下文，初始化前处理，（之后的操作根据是否实现接口而调用）->**自定义的初始化方法**->初始化后处理方法->bean可以使用，驻留在应用上下文中-destroy()方法

 - BeanNameAware接口，Spring将**bean的ID**传递给setBean-Name()方法
 - BeanFactoryAware接口，Spring将调用setBeanFactory()方法，将**BeanFactory容器实例**传入；
 - ApplicationContextAware接口，Spring将调用setApplicationContext()方法，将bean所在的**应用上下文的引用**传入进来；
 - BeanPostProcessor接口，Spring将调用它们的post-ProcessBeforeInitialization()方法；
 - InitializingBean接口，Spring将调用它们的after-PropertiesSet()方法。类似地，如果bean使用initmethod声明了初始化方法，该方法也会被调用；
 - BeanPostProcessor接口，Spring将调用它们的post-ProcessAfterInitialization()方法；
 - 了DisposableBean接口，Spring将调用它的destroy()接口方法。

# 第二章 装配Bean

创建应用对象之间协作关系的行为通常称为装配（wiring），这也是
依赖注入（DI）的本质。

三种机制：显式（XML，Java），隐式（bean发现机制和自动装载）

## 自动装配Bean
**@Component注解**。这个简单的注解表明该类会作为组件类，并告知Spring要为这个类创建bean。

**组件扫描：**
不过，组件扫描默认是不启用的。我们还需要**显式配置一下Spring**，从而命令它去寻找带有@Component注解的类，并为其创建bean。
1.**@ComponentScan注解**启用了组件扫描，添加在类上。
2.XMl使用Spring context命名空间的```<context:component-scan>```元素。
如：```<context:component-scan base-package="soundsystem">```